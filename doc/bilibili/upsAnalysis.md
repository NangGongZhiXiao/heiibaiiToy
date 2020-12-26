# 分析你在B站关注的UP主

## 使用方式

1. 打开[bilibili.com](https://www.bilibili.com/)
2. 登录自己的账号
3. 按F12打开控制台
4. 复制下面的代码到控制台然后点击回车，之后就是耐心等待
```
const UPS_URL = "https://api.bilibili.com/x/relation/followings?ps=20&order=desc&jsonp=jsonp"
const MY_INFO_URL = "https://api.bilibili.com/x/space/myinfo?jsonp=jsonp"
const UP_STAT_URL= "https://api.bilibili.com/x/relation/stat?jsonp=jsonp"
const UP_UPSTAT_URL= "https://api.bilibili.com/x/space/upstat?jsonp=jsonp"
var myFollowings = []
var my = {}

function getMyInfo() {
  return new Promise((resolve, reject) => {
    $.ajax({
			type: "get",
			url: MY_INFO_URL,
			dataType: "jsonp",
			success: function(data) {
        resolve(data)
      },
      error: function(e) {
        reject(e)
      }
		})
  })
}

function getUps(id, page) {
  return new Promise((resolve, reject) => {
    $.ajax({
			type: "get",
			url: UPS_URL+`&vmid=${id}&pn=${page}`,
			dataType: "jsonp",
			success: function(data) {
        resolve(data)
      },
      error: function(e) {
        reject(e)
      }
    })
  })
}

function getUpStat(id) {
  return new Promise((resolve, reject) => {
    $.ajax({
			type: "get",
			url: UP_STAT_URL+`&vmid=${id}`,
			dataType: "jsonp",
			success: function(data) {
        resolve(data)
      },
      error: function(e) {
        reject(e)
      }
    })
  })
}

function getUpUPStat(id) {
  return new Promise((resolve, reject) => {
    $.ajax({
			type: "get",
			url: UP_UPSTAT_URL+`&mid=${id}`,
			dataType: "jsonp",
			success: function(data) {
        resolve(data)
      },
      error: function(e) {
        reject(e)
      }
    })
  })
}

getMyInfo().then(data=>{
  my = data.data
  myFollowings = []
  return getUps(data.data.mid, 1)
}).then(data=>{
  const total = data.data.total
  myFollowings.push(...data.data.list)
  const pages = []
  if(total<=20) return [{data: {list: []}}]
  for(let i = 2; i <= Math.ceil((total-20)/20)+1; i++) {
    pages.push(getUps(my.mid, i))
  }
  return Promise.all(pages)
}).then(data=>{
  data.forEach(i=>{
    myFollowings.push(...i.data.list)
  })
  const total = myFollowings.length
  return Promise.all(myFollowings.map((i, index)=>new Promise((resolve, reject)=>{
    setTimeout(() => {
      Promise.all([getUpStat(i.mid), getUpUPStat(i.mid)]).then(data=>{
        console.clear()
        console.log("为了防止请求过快被封IP，放慢了请求速度，请耐心等待");
        console.log(`......正在获取数据，预计剩余${total-index}秒`);
        myFollowings[index].stat = data[0].data
        myFollowings[index].upstat = data[1].data
        resolve()
      }, reject)
    }, index*1000);
  })))
}).then(() => {
  console.table(myFollowings.map(i=>{
    return {
      "up": i.uname,
      "签名": i.sign,
      "认证": i.official_verify.desc,
      "粉丝数": i.stat.follower,
      "播放数": i.upstat.archive.view,
      "获赞数": i.upstat.likes
    }
  }))
})
```
你还可以点击控制台上的表头来对up主根据粉丝数、播放数、获赞数进行排序

## 原理解释

正在编辑......