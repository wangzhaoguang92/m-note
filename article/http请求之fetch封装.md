# http请求之fetch封装

* 技术点：Fetch，URLSearchParams，ES7 async/await

```javascript
// import qs from 'qs'
// import 'whatwg-fetch'
import fetch from 'isomorphic-fetch'
import URLSearchParams from 'url-search-params'

/**
 * 序列化参数 (get)
 * @param {any} obj
 * @returns
 */
function serialiseObject (obj) {
  const prefix = '?'

  if (obj && Object.keys(obj).length) {
    return prefix + Object.keys(obj).map(key =>
      `${key}=${encodeURIComponent(obj[key])}`
    ).join('&')
  }

  return ''
}

/**
 * 获取参数 (post)
 * @param {any} payload
 * @returns
 */
function getParams (payload) {
  let datas = new URLSearchParams()
  for (const prop in payload) {
    let val = typeof payload[prop] === 'object'
      ? JSON.stringify(payload[prop])
      : payload[prop]
    datas.set(prop, val)
  }
  // console.log(datas.toString())
  return datas
}

/**
 * 检查状态码
 * @param {any} response
 * @returns
 */
function checkStatus (response) {
  if (response.status >= 200 && response.status < 300) {
    return response
  } else {
    var error = new Error(response.statusText)
    error.response = response
    throw error
  }
}

/**
 * 返回数据
 * @param {any} response
 * @returns
 */
function parseJSON (response) {
  return response.json()
}

const http = {
  /**
   * get 请求方式
   * @param {any} path
   * @param {any} params
   * @returns
   */
  async get (path, params) {
    let url = `${path}${serialiseObject(params)}`
    try {
      let request = await fetch(url)
      let checked = await checkStatus(request)
      let json = await parseJSON(checked)

      return json
    } catch (error) {
      throw new Error(error)
    }
  },

  /**
   * post请求方式
   * @param {any} url
   * @param {any} payload
   * @returns
   */
  async post (url, payload) {
    // let options = {
    //   method: 'POST',
    //   body: qs.stringify(body),
    //   headers: {
    //     'Content-Type': 'application/x-www-form-urlencoded'
    //   }
    // }
    let options = {
      method: 'POST',
      body: getParams(payload)
    }
    try {
      let request = await fetch(url, options)
      let checked = await checkStatus(request)
      let json = await parseJSON(checked)

      return json
    } catch (error) {
      throw new Error(error)
    }
  }

}

export default http
```