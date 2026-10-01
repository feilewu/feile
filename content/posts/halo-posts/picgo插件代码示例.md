---
title: picgo插件代码示例
id: 88
date: 2024-09-27 19:10:53
auther: admin
cover: 
excerpt: 
permalink: /?p=88
categories:
tags: 
 - picgo
---

```javascript
import picgo from 'picgo'
// @ts-ignore
import { PluginConfig } from 'picgo/dist/utils/interfaces'


const config = (ctx: picgo): PluginConfig[] => {
  let userConfig = ctx.getConfig('picBed.imgYun')
  if (!userConfig) {
    userConfig = {}
  }
  return [
    {
      name: 'serverHost',
      type: 'input',
      default: userConfig.serverHost || '',
      message: 'serverHost 不能为空',
      required: true
    },
    {
      name: 'apiToken',
      type: 'password',
      default: userConfig.apiToken || '',
      message: 'apiToken 不能为空',
      required: true
    }
  ]
}

const postOptions = (fileName:string, options: any, image: Buffer): any => {
  return {
    method: 'POST',
    url: options.serverHost+'/api/img/upload',
    headers: {
      contentType: 'multipart/form-data',
      'User-Agent': 'PicGo',
      'token': options.apiToken
    },
    formData: {
      file: {
        value: image,
        options: {
          filename: fileName
        }
      }
    }
  }
}

const handle = async (ctx: picgo): Promise<picgo> => {
  const imgYunOptions = ctx.getConfig('picBed.imgYun')
  if (!imgYunOptions) {
    throw new Error('imgYun 图床配置文件')
  }
  try {
    const imgList = ctx.output
    for (const img of imgList) {
      let image = img.buffer
      if (!image && img.base64Image) {
        image = Buffer.from(img.base64Image, 'base64')
      }
      const options = postOptions(img.fileName,imgYunOptions,image);
      let body = await ctx.Request.request(options)
      body = JSON.parse(body)
      if (body.success) {
        delete img.base64Image
        delete img.buffer
        img.imgUrl = body.obj
      } else {
        ctx.log.error(body)
        throw new Error('Server error, please try again')
      }

    }
    return ctx
  } catch (err) {
    throw err
  }
}

export = (ctx: picgo) => {
  const register = () => {
    ctx.helper.uploader.register('imgYun', {
      handle,
      name: 'imgYun',
      config: config
    })
  }
  return {
    uploader: 'imgYun',
    register
  }

}

```
