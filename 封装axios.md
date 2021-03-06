# 封装axios

>send.js

```
import Vue from "vue";
import { Message } from "element-ui";
import axios from "axios";
import qs from "qs";
axios.defaults.baseURL = "http://dev-api2.pingxiaobao.com";
axios.interceptors.request.use(
  function(config) {
    //在请求发出之前进行一些操作
    // config.headers = { "Content-Type": "multipart/form-data" };
    if (config.method === "post") {
      config.data = qs.stringify(config.data);
    }
    return config;
  },
  function(err) {
    return Promise.reject(err);
  }
);
//添加一个响应拦截器
axios.interceptors.response.use(
  function(res) {
    return res;
  },
  function(err) {
    return Promise.reject(err);
  }
);

var DataToString = function(data) {
  let str = "{";
  for (let key in data) {
    str += '"' + key + '":"' + data[key] + '",';
  }
  str = str.slice(0, str.length - 1);
  str += "}";
  return str;
};

var Login = function(url, data) {
  let keyMap = {
    appid: "U2YSLwZFRQsr5yBH",
    appkey: "ngywG2T3eCXVYtsr",
    payload: data,
    sequence: new Date().getTime(),
    timestamp: new Date().getTime(),
    uuid: getUuid(),
    version: "1.3" + new Date().getTime()
  };
  let singString = queryString(keyMap);
  let sign = md5(singString);
  delete keyMap.appkey;
  delete keyMap.payload;
  return url + "?" + queryString(keyMap) + "&sign=" + sign;
};

var Send = function(url, data) {
  let keyMap = {
    appid: "U2YSLwZFRQsr5yBH",
    appkey: "ngywG2T3eCXVYtsr",
    payload: data,
    sequence: new Date().getTime(),
    timestamp: new Date().getTime(),
    token: getToken(),
    uuid: getUuid(),
    version: "1.3" + new Date().getTime()
  };
  let singString = queryString(keyMap);
  let sign = md5(singString);
  delete keyMap.appkey;
  delete keyMap.payload;
  return url + "?" + queryString(keyMap) + "&sign=" + sign;
};

var getToken = function() {
  let token;
  try {
    token = localStorage.getItem("token");
  } catch (error) {}
  if (token && token != "undefined") {
    return token;
  } else {
    return null;
  }
};

var queryString = function(keyMap) {
  let result = "";
  for (let element in keyMap) {
    result += element + "=" + keyMap[element] + "&";
  }
  return result.substring(0, result.length - 1);
};

//获取与生成uuid
var getUuid = function() {
  let uuid;
  const guid = function() {
    return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function(c) {
      let r = (Math.random() * 16) | 0,
        v = c == "x" ? r : (r & 0x3) | 0x8;
      return v.toString(16);
    });
  };
  try {
    uuid = localStorage.getItem("uuid");
  } catch (error) {}

  if (uuid && uuid != "undefined") {
    return uuid;
  } else {
    uuid = guid();
    try {
      localStorage.setItem("uuid", uuid);
    } catch (error) {}
    return uuid;
  }
};
// 获取环境
var getEnv = function() {
  if (
    window.location.href.indexOf("localhost") >= 0 ||
    window.location.href.indexOf("172.") >= 0 ||
    window.location.href.indexOf("192.168.") >= 0
  ) {
    return "dev-";
  }
  var pattern = /(dev-|tst-|uat-)/i;
  var matcher = window.location.href.match(pattern);
  if (matcher && matcher.length >= 2) {
    return matcher[1];
  }
  return "";
};

const login = function(url, data) {
  let datas = {
    payload: DataToString(data)
  };
  return new Promise((resolve, reject) => {
    axios
      .post(Login(url, datas.payload), datas, {
        headers: { "Content-Type": "application/x-www-form-urlencoded" }
      })
      .then(function(res) {
        resolve(res.data);
      })
      .catch(function(res) {
        reject(res.data);
      });
  });
};

const send = function(url, data) {
  let datas = { payload: DataToString(data) };
  return new Promise((resolve, reject) => {
    axios
      .post(Send(url, datas.payload), datas, {
        headers: { "Content-Type": "application/x-www-form-urlencoded" }
      })
      .then(function(res) {
        resolve(res.data);
      })
      .catch(function(res) {
        Message({
          message: res.data.message,
          type: "error",
          duration: 3000
        });
        reject(res.data);
      });
  });
};
const http = {
  login,
  send
};
export default http;

    
```

