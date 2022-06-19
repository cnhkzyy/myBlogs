[toc]





# 开发平台



## 登录



### 前端代码

#### 1. @/api/user.js

```javascript
import request from '@/utils/request'

export function login(data) {
  return request({
    url: '/users/login/',
    method: 'post',
    data
  })
}

export function getInfo() {
  return request({
    url: '/users/info',
    method: 'get',
  })
}
```

#### 2. @/store/modules/user.js

```javascript
const actions = {
  // user login
  login({ commit }, userInfo) {
    const { username, password } = userInfo
    return new Promise((resolve, reject) => {
      login({ username: username.trim(), password: password }).then(response => {
        const { data } = response
        commit('SET_TOKEN', data.token)
        setToken(data.token)
        resolve()
      }).catch(error => {
        reject(error)
      })
    })
  },

  // get user info
  getInfo({ commit, state }) {
    return new Promise((resolve, reject) => {
      getInfo(state.token).then(response => {
        const { data } = response

        if (!data) {
          reject('Verification failed, please Login again.')
        }

        const { roles, name, avatar, introduction } = data

        // roles must be a non-empty array
        if (!roles || roles.length <= 0) {
          reject('getInfo: roles must be a non-null array!')
        }

        commit('SET_ROLES', roles)
        commit('SET_NAME', name)
        commit('SET_AVATAR', avatar)
        commit('SET_INTRODUCTION', introduction)
        resolve(data)
      }).catch(error => {
        reject(error)
      })
    })
  },
```

#### 3. @/views/login/index

```javascript
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true
            user.login(this.loginForm)
          // this.$store.dispatch('user/login', this.loginForm)
            .then(res => {
              sessionStorage.clear()
              localStorage.token = res.token
              localStorage.user_id = res.user_id
              localStorage.username = res.username
              this.$router.push({ path: this.redirect || '/', query: this.otherQuery })
              this.loading = false
            })
            .catch(() => {
              this.loading = false
            })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    }
```

#### 4. @/utils/auth.js

```javascript
export function getToken() {
  //return Cookies.get(TokenKey)
  return localStorage.token
}
```

#### 5. @src/permisson.js

```javascript
router.beforeEach(async(to, from, next) => {
  // start progress bar
  NProgress.start()

  // set page title
  document.title = getPageTitle(to.meta.title)

  // determine whether the user has logged in
  const hasToken = getToken()
```

#### 6.  @/utils/request.js

```javascript
import axios from 'axios'
import { MessageBox, Message } from 'element-ui'
import store from '@/store'
import { getToken } from '@/utils/auth'

// create an axios instance
const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API, // url = base url + request url
  // withCredentials: true, // send cookies when cross-domain requests
  timeout: 5000 // request timeout
})

// request interceptor
service.interceptors.request.use(
  config => {
    // do something before request is sent
    const token = getToken()

    if (token) {
      // let each request carry token
      // ['X-Token'] is a custom headers key
      // please modify it according to the actual situation
      //config.headers['X-Token'] = getToken()
      config.headers.Authorization = `JWT ${token}`
    }
    return config
  },
  error => {
    // do something with request error
    console.log(error) // for debug
    return Promise.reject(error)
  }
)

// response interceptor
service.interceptors.response.use(
  /**
   * If you want to get http information such as headers or status
   * Please return  response => response
  */

  /**
   * Determine the request status by custom code
   * Here is just an example
   * You can also judge the status by HTTP Status Code
   */
  response => {
    const res = response

    // if the custom code is not 20000, it is judged as an error.
    if (res.status !== 200) {
      Message({
        message: res.message || 'Error',
        type: 'error',
        duration: 5 * 1000
      })

      // 50008: Illegal token; 50012: Other clients logged in; 50014: Token expired;
      if (res.code === 50008 || res.code === 50012 || res.code === 50014) {
        // to re-login
        MessageBox.confirm('You have been logged out, you can cancel to stay on this page, or log in again', 'Confirm logout', {
          confirmButtonText: 'Re-Login',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          store.dispatch('user/resetToken').then(() => {
            location.reload()
          })
        })
      }
      return Promise.reject(new Error(res.message || 'Error'))
    } else {
      return res
    }
  },
  error => {
    console.log('err' + error) // for debug
    Message({
      message: error.message,
      type: 'error',
      duration: 5 * 1000
    })
    return Promise.reject(error)
  }
)

export default service
```

### 后端代码

#### 1. apps/users/urls.py

```
from rest_framework import routers
from rest_framework_jwt.views import obtain_jwt_token
from django.urls import path
from .views import *


urlpatterns = [
    path("login/", obtain_jwt_token),
    path("info/", getInfo),
]
```



#### 2. apps/users/views.py

```python
from rest_framework import viewsets
from django.http import JsonResponse

# Create your views here.
def getInfo(request):
    return JsonResponse({'roles': '[admin]',
                     'name': 'admin',
                     'avatar': 'https://wpimg.wallstcn.com/f778738c-e4f8-4870-b634-56703b4acafe.gif'})
```

#### 3. utils/jwt_handler.py

```python
def jwt_response_payload_handler(token, user = None, request = None):
    return {
        "token": token,
        "user_id": user.id,
        "username": user.username
    }
```



#### 4. settings.py

```python
JWT_AUTH = {
    # 默认5分钟过期，可以使用JWT_EXPIRATION_DELTA来设置过期时间为1天
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=1),
    # 默认的前缀是JWT，可以使用JWT_AUTH_HEADER_PREFIX修改前缀为B
    #'JWT_AUTH_HEADER_PREFIX': 'JWT',
    'JWT_RESPONSE_PAYLOAD_HANDLER': 'utils.jwt_handler.jwt_response_payload_handler',
}
```

