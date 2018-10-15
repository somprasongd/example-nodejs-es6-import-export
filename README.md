# example-nodejs-es6-import-export
Example how to use ES6 import/export in nodejs

สามารถทำได้หลายวิธี แต่ในตัวอย่างนี้จะใช้ 2 วิธี คือ

1. ใช้ [babel](https://babeljs.io/)
2. ใช้โมดูล [esm](https://www.npmjs.com/package/esm)

## แบบใช้ Babel

- เริ่มจากสร้างโปรเจค `mkdir node-es6-babel && cd $_`
- สร้างไฟล์ package.json `npm init -y`
- ติดตั้ง babel `npm i -D @babel/cli @babel/core @babel/node @babel/preset-env`
- สร้างไฟล์ .babelrc (`touch .babelrc`) แล้วตั้งค่าตามนี้
```json
{
  "presets": [
    [
      "env",
      {
        "targets": {
          "node": "current"
        }
      }
    ]
  ]
}
```
**หมายเหตุ** `"node": "current"` คือ ระบุ version ของ nodejs ที่ใช้รัน babel (เหมือน `"node": process.versions.node`)

- สร้างไฟล์ src/index.js และไฟล์ src/my-module.js

```js
// src/my-module.js
// Named export - Has a name. Have as many as needed.
export const message = 'Some message from my-module.js';

// Default export - Has no name. You can only have one.
export default (name) => `Hello ${name}`;

// or
// const message = 'Some message from my-module.js';
// const getGreeting = (name) => `Hello ${name}`;
// export { message, getGreeting as default }
```

```js
// src/index.js
import greetingFn, { message } from './my-module';

console.log(message);

console.log(greetingFn('Ball'));
```

- แก้ไข script ในไฟล์ package.json ตามนี้

```json
{
  "scripts": {
    "start": "babel-node src/index.js"
  },
}
```

หรือถ้าใช้ร่วมกับ `nodemon`

```json
{
  "scripts": {
    "start": "nodemon src/index.js --exec babel-node -e js"
  },
}
```

- ทดสอบรันดู `npm run start`

```bash
Some message from my-module.js
Hello Ball
```

- เมื่อจะเอาโค้ดไป deploy ต้อง build ก่อน ทำได้โดยเพิ่ม scripts ใน package.json ดังนี้
```json
{
  "scripts": {
    "start": "babel-node src/index.js",
    "clean": "rm -rf dist",
    "build": "npm run clean && mkdir dist && babel src -s -d dist --copy-files",
    "prod": "export NODE_ENV=production||set NODE_ENV=production&&node dist/index.js",
  },
}
```

- build โดยคำสั่ง `npm run build` จะได้ folder dist ออกมา
- รันโดยใช้คำสั่ง `npm run prod`

## แบบใช้ NPM module ESM

- เริ่มจากสร้างโปรเจค `mkdir node-es6-esm && cd $_`
- สร้างไฟล์ package.json `npm init -y`
- ติดตั้ง esm `npm i esm`
- สร้างไฟล์ src/index.js, src/main.js และไฟล์ src/my-module.js

```js
// src/my-module.js
// Named export - Has a name. Have as many as needed.
export const message = 'Some message from my-module.js';

// Default export - Has no name. You can only have one.
export default (name) => `Hello ${name}`;

// or
// const message = 'Some message from my-module.js';
// const getGreeting = (name) => `Hello ${name}`;
// export { message, getGreeting as default }
```

```js
// src/main.js
import greetingFn, { message } from './my-module';

console.log(message);

console.log(greetingFn('Ball'));
```

```js
// src/index.js
require = require('esm')(module);
module.exports = require('./main.js');
```

- แก้ไข script ในไฟล์ package.json ตามนี้

```json
{
  "scripts": {
    "start": "node ./src/index.js",
    "dev": "nodemon ./src/index.js",
    "prod": "export NODE_ENV=production||set NODE_ENV=production&&npm start"
  },
}
```

หรือถ้าใช้ร่วมกับ `nodemon`

```json
{
  "scripts": {
    "start": "node ./src/index.js",
    "dev": "nodemon ./src/index.js",
    "prod": "export NODE_ENV=production||set NODE_ENV=production&&npm start"
  },
}
```

- ทดสอบรันดู `npm run start` หรือ `npm run dev`

```bash
Some message from my-module.js
Hello Ball
```

- เมื่อจะเอาโค้ดไป deploy แบบนี้ไม่ต้อง build สามารถรันได้เลยโดยใช้คำสั่ง `npm run prod` 