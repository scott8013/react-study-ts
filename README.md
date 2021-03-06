## 创建工程

```shell
npx create-react-app [ProjectName] --template typescript
```

## 安装 prettier

```shell
yarn add prettier -D
```

### 根目录增加配置文件 .prettierrc

```
{
 "trailingComma": "es5",
 "tabWidth": 2,
 "semi": false,
 "singleQuote": true
}
```

## 安装 eslint

```shell
yarn add eslint -D

npx eslint --init //初始化Eslint配置文件
```

## This section also starts with installing an npm package called eslint-plugin-prettier, which will help us configure eslint and prettier together. We’ll install it with

```shell
yarn add eslint-plugin-prettier -D
```

```.eslintrc.js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: ['plugin:react/recommended', 'airbnb'],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  plugins: ['react', '@typescript-eslint', 'prettier'],
  rules: {
    semi: 0,
    'comma-dangle': 0,
    'prettier/prettier': 'error',
    'react/jsx-filename-extension': [
      1,
      { extensions: ['.ts', '.tsx', '.js', 'jsx'] },
    ],
    'react/jsx-one-expression-per-line': 0,
    'no-unused-vars': 0,
    'object-curly-newline': 0,
    'import/extensions': 0,
    'import/no-unresolved': 0,
  },
}

```

## Also, let’s add two scripts to our package.json file. This will help us to lint files by npm run lint and format our code by npm run pretty

```package.json
 "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "lint": "eslint --fix",
    "pretty": "prettier --write ."
  }
```

## .eslintignore

```
node_modules
public
build
```

## .prettierignore

```
# Ignore artifacts:
build
coverage

# Ignore all HTML files:
*.html
```

## React 国家化部分

> 参考文档： https://dev.to/adrai/how-to-properly-internationalize-a-react-application-using-i18next-3hdb

### Step 1.0

> 安装依赖

```shell
yarn add i18next react-i18next i18next-browser-languagedetector i18next-http-backend
```

### Step 2.0

> 在 src 目录下创建文件夹 => i18n 在 i8n 文件中 新建文件 index.ts 内容如下

```ts
import i18n from 'i18next'
import { initReactI18next } from 'react-i18next'
import LanguageDetector from 'i18next-browser-languagedetector'
import Backend from 'i18next-http-backend'
import { DateTime } from 'luxon'

i18n
  // i18next-http-backend
  // loads translations from your server
  // https://github.com/i18next/i18next-http-backend
  .use(Backend)
  // detect user language
  // learn more: https://github.com/i18next/i18next-browser-languageDetector
  .use(LanguageDetector)
  // pass the i18n instance to react-i18next.
  .use(initReactI18next)
  // init i18next
  // for all options read: https://www.i18next.com/overview/configuration-options
  .init({
    debug: true,
    fallbackLng: 'en',
    interpolation: {
      escapeValue: false, // not needed for react as it escapes by default
    },
  })
  .then((v) => v)

// 格式化时间用的 不符合我的开发习惯 可以不用 luxon 也可以不引入
i18n.services.formatter.add('DATE_HUGE', (value, lng, options) => {
  return DateTime.fromJSDate(value)
    .setLocale(lng)
    .toLocaleString(DateTime.DATE_HUGE)
})

export default i18n
```

### Step 3.0

> 在 public 文件夹下新建文件夹 locales
> 在 index.tsx 中引入 /i8n/index.ts
> locales 新建文件夹 cn en de tw 等
> cn 中新增文件 translation.json

```json
{
  "description": {
    "part1": "Hello world, Cn",
    "part2": "Hello world, Cn, I'm Learn React"
  },
  "counter": "Changed language already {{count}} times-CN",
  "footer": {
    "date": "Today is {{date, DATE_HUGE}}",
    "date_morning": "Good morning! Today is {{date, DATE_HUGE}} | Have a nice day!",
    "date_afternoon": "Good afternoon! It's {{date, DATE_HUGE}}",
    "date_evening": "Good evening! Today was the {{date, DATE_HUGE}}"
  }
}
```

> 如果有多种语言 需要增加对应的翻译文件

### Step 4.0 Usage 如何使用 感觉经验 满足日常开发需求 并根据具体业务需求拆分组建

```tsx
import { Space } from 'antd'
import { useTranslation, Trans } from 'react-i18next'
import { useState } from 'react'

// eslint-disable-next-line no-undef
const getGreetingTime = (d = new Date()) => {
  const splitAfternoon = 12 // 24hr time to split the afternoon
  const splitEvening = 17 // 24hr time to split the evening
  const currentHour = parseFloat(String(d.getHours()))

  if (currentHour >= splitAfternoon && currentHour <= splitEvening) {
    return 'afternoon'
  }
  if (currentHour >= splitEvening) {
    return 'evening'
  }
  return 'morning'
}

const lngs = {
  // en_US: { nativeName: 'English' },
  // de_DE: { nativeName: 'Deutsch' },
  // zh_CN: { nativeName: 'China' },
  // zh_TW: { nativeName: 'China' },
  en: { nativeName: 'English' },
  de: { nativeName: 'Deutsch' },
  cn: { nativeName: 'China-CN' },
  tw: { nativeName: 'China-TW' },
}
function I18n() {
  const { t, i18n } = useTranslation()
  const [count, setCount] = useState(0)
  return (
    <div className="I18n">
      <p>
        <Trans i18nKey="description.part1">
          {/* Edit1 <1>src/App.js</1> and save to reload2. */}
          Edit <code>src/I18n.js</code> and save to reload.
        </Trans>
      </p>
      <p>{t('description.part2')}</p>

      <hr />

      <div>
        <Space>
          {Object.keys(lngs).map((lng) => (
            <button
              key={lng}
              style={{
                fontWeight: i18n.resolvedLanguage === lng ? 'bold' : 'normal',
              }}
              type="submit"
              onClick={() => {
                setCount(count + 1)
                return i18n.changeLanguage(lng)
              }}
            >
              {lngs[lng].nativeName}
            </button>
          ))}
        </Space>
        <hr />
        {/*  Interpolation and Pluralization */}
        <p>
          <i>{t('counter', { count })}</i>
        </p>
        <hr />
        <div>{t('footer.date', { date: new Date() })}</div>
        <hr />
        <div className="Footer">
          <div>
            {t('footer.date', { date: new Date(), context: getGreetingTime() })}
          </div>
        </div>
      </div>
    </div>
  )
}

export default I18n
```

### Step 5.0 通过下面方法获取当前的语言 做个映射 方便引入对应 antd 语言

```tsx
const i18nextLng = localStorage.getItem('i18nextLng')

// eslint-disable-next-line no-shadow
enum Lngs {
  en = 'en_US',
  tw = 'zh_TW',
  zh = 'zh_CN',
  de = 'de_DE',
}

console.log(Lngs[i18nextLng], Lngs, '当前语言映射')
```

## 配置路径别名

### Step 01. tsconfig.json 增加如下代码

```json
{
  "baseUrl": "./",
  "paths": {
    "@/*": ["src/*"],
    "utils": ["src/utils/"]
  }
}
```

### Step 02. craco.config.js 增加如下配置 相信配置请查阅： https://www.npmjs.com/package/@craco/craco

```
  webpack: {
   ...
    alias: {
      '@': resolve(__dirname, './src'),
      utils: resolve(__dirname, './src/utils')
    }
  }
```
