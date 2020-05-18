
# Example: use Oak to build a REST API


## 1.新建一个 `app.ts` 文件

```
import { Application, Router } from 'https://deno.land/x/oak/mod.ts'

const env = Deno.env.toObject()
const PORT = env.PORT || 4000
const HOST = env.HOST || '127.0.0.1'

const router = new Router()

const app = new Application()

app.use(router.routes())
app.use(router.allowedMethods())

console.log(`Listening on port ${PORT}...`)

await app.listen(`${HOST}:${PORT}`)
```

## 2.运行 deno

```
deno run --allow-env --allow-net app.ts
```

这时候会下载依赖，监听 4000 端口；


## 3.定义接口

```
interface Dog {
  name: string
  age: number
}

let dogs: Array<Dog> = [
  {
    name: 'Roger',
    age: 8,
  },
  {
    name: 'Syd',
    age: 7,
  },
]
```

## 4.实现路由

```
const router = new Router()

router
  .get('/dogs', getDogs)
  .get('/dogs/:name', getDog)
  .post('/dogs', addDog)
  .put('/dogs/:name', updateDog)
  .delete('/dogs/:name', removeDog)
```

## 5.实现 `GET /dogs`，这个会返回所有的集合 ;

```
export const getDogs = ({ response }: { response: any }) => {
  response.body = dogs
}

```

## 6.通过名字获取某一个

```
export const getDog = ({
  params,
  response,
}: {
  params: {
    name: string
  }
  response: any
}) => {
  const dog = dogs.filter((dog) => dog.name === params.name)
  if (dog.length) {
    response.status = 200
    response.body = dog[0]
    return
  }

  response.status = 400
  response.body = { msg: `Cannot find dog ${params.name}` }
}
```

## 7.实现添加 ``

```
export const addDog = async ({
  request,
  response,
}: {
  request: any
  response: any
}) => {
  const body = await request.body()
  const dog: Dog = body.value
  dogs.push(dog)

  response.body = { msg: 'OK' }
  response.status = 200
}
```

## 8.实现更新

```
export const updateDog = async ({
  params,
  request,
  response,
}: {
  params: {
    name: string
  }
  request: any
  response: any
}) => {
  const temp = dogs.filter((existingDog) => existingDog.name === params.name)
  const body = await request.body()
  const { age }: { age: number } = body.value

  if (temp.length) {
    temp[0].age = age
    response.status = 200
    response.body = { msg: 'OK' }
    return
  }

  response.status = 400
  response.body = { msg: `Cannot find dog ${params.name}` }
}
```

## 9.实现删除

```
export const removeDog = ({
  params,
  response,
}: {
  params: {
    name: string
  }
  response: any
}) => {
  const lengthBefore = dogs.length
  dogs = dogs.filter((dog) => dog.name !== params.name)

  if (dogs.length === lengthBefore) {
    response.status = 400
    response.body = { msg: `Cannot find dog ${params.name}` }
    return
  }

  response.body = { msg: 'OK' }
  response.status = 200
}
```

## 完整代码

```
import { Application, Router } from 'https://deno.land/x/oak/mod.ts'

const env = Deno.env.toObject()
const PORT = env.PORT || 4000
const HOST = env.HOST || '127.0.0.1'

interface Dog {
  name: string
  age: number
}

let dogs: Array<Dog> = [
  {
    name: 'Roger',
    age: 8,
  },
  {
    name: 'Syd',
    age: 7,
  },
]

export const getDogs = ({ response }: { response: any }) => {
  response.body = dogs
}

export const getDog = ({
  params,
  response,
}: {
  params: {
    name: string
  }
  response: any
}) => {
  const dog = dogs.filter((dog) => dog.name === params.name)
  if (dog.length) {
    response.status = 200
    response.body = dog[0]
    return
  }

  response.status = 400
  response.body = { msg: `Cannot find dog ${params.name}` }
}

export const addDog = async ({
  request,
  response,
}: {
  request: any
  response: any
}) => {
  const body = await request.body()
  const { name, age }: { name: string; age: number } = body.value
  dogs.push({
    name: name,
    age: age,
  })

  response.body = { msg: 'OK' }
  response.status = 200
}

export const updateDog = async ({
  params,
  request,
  response,
}: {
  params: {
    name: string
  }
  request: any
  response: any
}) => {
  const temp = dogs.filter((existingDog) => existingDog.name === params.name)
  const body = await request.body()
  const { age }: { age: number } = body.value

  if (temp.length) {
    temp[0].age = age
    response.status = 200
    response.body = { msg: 'OK' }
    return
  }

  response.status = 400
  response.body = { msg: `Cannot find dog ${params.name}` }
}

export const removeDog = ({
  params,
  response,
}: {
  params: {
    name: string
  }
  response: any
}) => {
  const lengthBefore = dogs.length
  dogs = dogs.filter((dog) => dog.name !== params.name)

  if (dogs.length === lengthBefore) {
    response.status = 400
    response.body = { msg: `Cannot find dog ${params.name}` }
    return
  }

  response.body = { msg: 'OK' }
  response.status = 200
}

const router = new Router()
router
  .get('/dogs', getDogs)
  .get('/dogs/:name', getDog)
  .post('/dogs', addDog)
  .put('/dogs/:name', updateDog)
  .delete('/dogs/:name', removeDog)

const app = new Application()

app.use(router.routes())
app.use(router.allowedMethods())

console.log(`Listening on port ${PORT}...`)

await app.listen(`${HOST}:${PORT}`)
```