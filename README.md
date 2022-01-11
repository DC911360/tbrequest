# tbrequest 
```js

var app = getApp()
let path = ''

/*
  属性：
    userAgentTb：UA
    formatDateUTC：服务器时间戳
    authorization：接口需要的验签
    domain：接口的域名
    showToast：toast提示
    report_noaction：mattermost 通知
    report_default：mattermost 通知
    fName：框架名称
*/

function reqObject(argument){
  this.userAgentTb = argument.userAgentTb;
  this.formatDateUTC = argument.formatDateUTC;
  this.authorization = argument.authorization;
  this.domain = argument.domain;
  this.showToast = argument.showToast;
  this.report_noaction = argument.report_noaction;
  this.report_default =  argument.report_default;
  this.fName = argument.fName


  //判断domain + url 的函数
  this.request = (data)=>{
    // console.log("data:",data)
    return new Promise((re, rj) => {
      if (data.apiType == "api") {
        path = this.domain.api
      }
      if (data.apiType == "mp") {
        path = this.domain.mp
      }
      data.path = path;
      console.log('data：', data);
      if (data.data.requestType == "login" || data.data.requestType == "reportNoaction" || data.data.requestType == "reportDefault") {
        this.publicRequest(data).then(
          res => {
            re(res)
          },
          err => {
            rj(err)
          })
      } else {
        if (!getApp().globalData.uc_uid) {
          getApp().userLogin('', 'page').then((res) => {
            data.data.uc_uid = getApp().globalData.uc_uid;
            this.publicRequest(data).then(
              res => {
                re(res)
              },
              err => {
                rj(err)
              })
          }, (err) => {
            if (err.errMsg == "request:fail timeout") {
              showToast({
                msg: '网络出现故障，请稍后重试'
              })
            } else {
              console.log("err", err)
            }
            rj('fail')
          })
        } else {
          this.publicRequest(data).then(
            res => {
              re(res)
            },
            err => {
              rj(err)
            })
        }
      }
    })
  }
  //公共的函数
  this.publicRequest = data=> {
    return new Promise((re, rj) => {
      let userAgentTb =this.userAgentTb
      let _date = this.formatDateUTC();
      let contentType = 'application/json; charset=UTF-8';
      let header={
        'content-type': contentType,
          'user-agent-tb': userAgentTb,
          'X-Tb-Date': _date,
          'Date': _date,
          'Authorization': this.authorization(_date, data.url, contentType)
      } 
      this.fName.request({
        url: data.path + data.url,
        data: data.data,
        header,
        method: data.method,
        dataType: 'json',
        responseType: 'text',
        success: (res) => {
          if (res.statusCode == 200) {
            re(res.data)
          } else {
            data.source = 'mini_program'
            data.getUid = true;
  
            if (data.data.count == 2) {
              return;
            }
            this.reportNoaction({
              page: 'http Error ',
              url: data.url,
              code: res.statusCode,
              input_params: data,
              type: 'reportNoaction',
              source: 'mini_program',
              count: 2
            })
          }
        },
        fail: (res) => {
          if (res.errMsg == "request:fail timeout") {
            showToast({
              msg: '网络出现故障，请稍后重试'
            })
          } else {
            console.log("res", res)
          }
  
          rj(res)
        },
        complete() {
  
        }
      })
    })
  }

}


module.exports = {
  reqObject
}


```
