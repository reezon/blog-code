---
title: 如何在dva的effects中实现回调
date: 2018-10-15 14:16:24
tags: dva redux react
---
### 需求
在前端实现修改用户密码的功能，通过dispatch来实现与后端api接口的交互。

界面点击“提交”执行的代码如下：
``` js
handleSubmit = () => {
  const { form, dispatch } = this.props

  form.validateFields((err, fieldsValue) => {
    if (err) return

    const values = {
      ...fieldsValue
    }

    dispatch({
      type: 'account/modifyPwd',
      payload: values
    })
  })
}
```
model中effect的代码如下：
``` js
* modifyPwd ({ payload }, { call, put }) {
  const response = yield call(apiMethod, payload)
}
```
此时前端需要获取到调用接口的结果，来显示“修改成功”or“修改失败”，

### 方法一：自己构造Promise
由于dva的1.x版本并未实现callback，只能通过自己构造Promise来实现回调。

界面代码修改为：
``` js
handleSubmit = () => {
    const { form, dispatch } = this.props

    form.validateFields((err, fieldsValue) => {
      if (err) return

      const values = {
        ...fieldsValue
      }

      new Promise((resolve) => {
        dispatch({
          type: 'account/modifyPwd',
          payload: {
            data: values,
            resolve
          }
        }).then((res) => {
          if (res) {
            message.success('修改成功')
          } else {
            message.error('修改失败')
          }
        })
      })

    })
  }
```

model中的effect代码修改为：
``` js
* modifyPwd ({ payload }, { call, put }) {
  const { data, resolve } = payload
  const response = yield call(apiMethod, data)
  !!resolve && resolve(response)
}
```

### 方法二：优雅的callback写法
在dva 2.x下，有另外一种callback的优雅的写法。

界面代码修改为：
``` js
handleSubmit = () => {
  const { form, dispatch } = this.props

  form.validateFields((err, fieldsValue) => {
    if (err) return

    const values = {
      ...fieldsValue
    }

    dispatch({
      type: 'account/modifyPwd',
      payload: values,
      callback: (response) => {
        if (response) {
          message.success('修改成功')
        } else {
          message.error('修改失败')
        }
      }
    })
  })
}
```

model中的effect代码修改为：
``` js
* modifyPwd ({ payload, callback }, { call, put }) {
  const response = yield call(apiMethod, payload)
  if (callback && typeof callback === 'function') {
      callback(response)
  }
}
```

### 方法三：使用then方法
在dva 2.x下，默认对dispatch提供了Promise回调，但是并未提供传参逻辑，因此需要通过model中的state来实现传参。

界面代码修改为：
``` js
handleSubmit = () => {
  const { form, dispatch, account } = this.props

  form.validateFields((err, fieldsValue) => {
    if (err) return

    const values = {
      ...fieldsValue
    }

    dispatch({
      type: 'account/modifyPwd',
      payload: values
    }).then(() => {
      if (account.modifyState) {
        message.success('修改成功')
      } else {
        message.error('修改失败')
      }
    })
  })
}
```

model中的effect代码修改为：
``` js
* modifyPwd ({ payload }, { call, put }) {
  const response = yield call(apiMethod, payload)
  yield put({
    type: 'saveModifyState',
    payload: {
      response: response
    }
  })
}
```
在model中实现一个reducer方法来修改state中的modifyState的状态：
``` js
saveModifyState (state, { payload }) {
  return {
    ...state,
    modifyState: payload.response
  }
}
```

### 总结
* 方法一在dva 1.x下是唯一的实现方式，在2.x之后就不再需要这么繁琐的写法了。
* 方法二通俗易懂，代码简洁，但是破坏了传统的redux的数据流转方式，不被布道者所提倡。
* 方法三是官方提倡的写法，但是需要多实现一个reducer，代码量比方法二要大。
