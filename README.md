# Apollo inside of NuxtJS

* Nuxt.js module to use [vue-apollo](https://github.com/Akryum/vue-apollo)
* uses internally same approach as [vue-cli-plugin-apollo](https://github.com/Akryum/vue-cli-plugin-apollo)

[![npm version](https://img.shields.io/npm/v/@nuxtjs/apollo.svg)](https://www.npmjs.com/package/@nuxtjs/apollo)
[![license](https://img.shields.io/github/license/nuxt-community/apollo-module.svg)](https://github.com/nuxt-community/apollo-module/blob/master/LICENSE)


## Setup

Install apollo module:

```bash
npm install --save @nuxtjs/apollo
# if you are using *.gql or *.graphql files add graphql-tag to your dependencies
npm install --save graphql-tag
```

Add `@nuxtjs/apollo` to `modules` section of `nuxt.config.js`

```bash
- clientConfigs: `Object` Config passed to ApolloClient
  - default: `Object`
  # alternative
  - default: `Path` // use this to have more control over the options
  - otherClient: `Object` (Optional)
```
```js
{
  // Add apollo module
  modules: ['@nuxtjs/apollo'],

  // Give apollo module options
  apollo: {
    tokenName: 'yourApolloTokenName', // optional, default: apollo-token
    tokenExpires: 10, // optional, default: 7
    includeNodeModules: true, // optional, default: false (this includes graphql-tag for node_modules folder)
    authenticationType: 'Basic', // optional, default: 'Bearer'
    // required
    clientConfigs: {
      default: {
        // required  
        httpEndpoint: 'http://localhost:4000',
        // optional
        // See https://www.apollographql.com/docs/link/links/http.html#options
        httpLinkOptions: {
          credentials: 'same-origin'
        },
        // You can use `wss` for secure connection (recommended in production)
        // Use `null` to disable subscriptions
        wsEndpoint: 'ws://localhost:4000', // optional
        // LocalStorage token
        tokenName: 'apollo-token', // optional
        // Enable Automatic Query persisting with Apollo Engine
        persisting: false, // Optional
        // Use websockets for everything (no HTTP)
        // You need to pass a `wsEndpoint` for this to work
        websocketsOnly: false // Optional
      },
      test: {
        httpEndpoint: 'http://localhost:5000',
        wsEndpoint: 'ws://localhost:5000',
        tokenName: 'apollo-token'
      },
      // alternative: user path to config which returns exact same config options
      test2: '~/plugins/my-alternative-apollo-config.js'
    }
  }
}
```

```js
// plugins/my-alternative-apollo-config.js
export default function(context){
  return {
    httpEndpoint: 'http://localhost:4000/graphql-alt'
    getAuth:() => 'Bearer my-static-token' // use this method to overwrite functions
  }
}
```

## Options
You can either (in a simple setup) just add an object as described above. If you need to overwrite cache or the default `getAuth()` function then use a path to your config file which returns the client config options.
### clientConfigs `Option`: required
Sets up the apollo client endpoints. All available options for each endpoint you find [here](https://github.com/Akryum/vue-cli-plugin-apollo/blob/master/graphql-client/src/index.js#L15)

Check out [official vue-apollo-cli](https://github.com/Akryum/vue-cli-plugin-apollo) where possible usecases are presented.

#### clientConfigs.default `Object`: required

#### clientConfigs.<your-additional-client-key> `Object|Path`: optional

### tokenName `String`: optional, default: 'apollo-token'

Token name for the cookie which will be set in case of authentication. You can also provide an option `tokenName` in each of your `clientConfigs` to overwrite the default.

### authenticationType `String`: optional, default: 'Bearer'

Sets the authentication type for any authorized request. Modify this if its not `Bearer`.

### includeNodeModules `Boolean`: optional, default: false

In case you use `*.gql` files inside of `node_module` folder you can enable the `graphql-tag/loader` to parse the files for you.

## Usage

Once the setup is completed you have a successfully enabled `vue-apollo` in your project. Checkout [Official example](https://github.com/nuxt/nuxt.js/tree/dev/examples/vue-apollo) and [vue-apollo](https://github.com/Akryum/vue-apollo) how to use `vue-apollo` inside your application code

## Authentication

You have following methods for authentication available:
```js
 // set your graphql-token
 this.$apolloHelpers.onLogin(token /* if not default you can pass in client as second argument, and you can set custom token expiration on third argument */)
 // unset your graphql-token
 this.$apolloHelpers.onLogout(/* if not default you can pass in client as second argument */)
 // get your current token (we persist token in a cookie)
 this.$apolloHelpers.getToken(/* you can provide named tokenName if not 'apollo-token' */)
```
Check out the [full example](https://github.com/nuxt-community/apollo-module/tree/master/test/fixture)

For permanent authorization tokens the setup just provide `getAuth` function for each of your endpoint configurations:
```js
  apollo: {
    clientConfigs: {
      default: {
        httpEndpoint: 'https://graphql.datocms.com',
        getAuth: () => 'Bearer your_token_string'
      },
    }
  },
```

#### User login
```js
methods:{
  async onSubmit () {
    const credentials = this.credentials
    try {
        const res = await this.$apollo.mutate({
            mutation: authenticateUserGql,
            variables: credentials
        }).then(({data}) => data && data.authenticateUser)
        await this.$apolloHelpers.onLogin(res.token)
    } catch (e) {
        console.error(e)
    }
  },
}
```
#### User logout
```js
methods:{
  async onLogout () {
    await this.$apolloHelpers.onLogout()
  },
}
```

#### getToken
```js
// middleware/isAuth.js
  export default function ({app, error}) {
    const hasToken = !!app.$apolloHelpers.getToken()
    if (!hasToken) {
        error({errorCode:503, message:'You are not allowed to see this'})
    }
}
```

#### Examples to access the defaultClient of your apolloProvider
##### Vuex actions
```js
export default {
  actions: {
    foo (store, payload) {
      let client = this.app.apolloProvider.defaultClient
    }
  }
}
```

##### asyncData/fetch method of page component
```js
export default {
  asyncData (context) {
    let client = context.app.apolloProvider.defaultClient
  }
}
```

##### onServerInit
```js
export default {
  nuxtServerInit (store, context) {
    let client = context.app.apolloProvider.defaultClient
  }
}
```


##### access client or call mutations of any method inside of component
```js
export default {
  methods:{
    foo(){
      // receive the associated Apollo client 
      const client = this.$apollo.getClient()

      // most likely you would call mutations like following:
      this.$apollo.mutate({mutation, variables})
    }
  }
}
```

##### query on any component
```js
export default {
  apollo: {
    foo: {
      query: fooGql,
      variables () {
        return {
          myVar: this.myVar
        }
      }
    }
  }
}
```

#### Add GQL file recognition on node_modules
```js
  apollo: {
    clientConfigs: {
      default: '~/apollo/client-configs/default.js'
    },
    includeNodeModules: true
  }
```

## Upgrade

### Upgrade Guide apollo-module v3 => v4

Version 4 of this module leaves you with zero configuration. This means we use the best possible approach available from `vue-cli-plugin-apollo` and the same configuration behaviour. This means you don't need to wire up your own configuration, simply pass 

Edit your configuration as following:
```js
// nuxt.config.js
apollo:{
 clientConfigs:{
  default:{
    httpEndpoint: YOUR_ENDPOINT,
    wsEndpoint: YOUR_WS_ENDPOINT
  }
 }
}
```

### Upgrade Guide apollo-client v1 => v2

Version 3 of this module is using apollo-client 2.x. You need to make sure to update all your middle/afterware according to the upgrade guide of apollo-client. Check this source for a reference: https://github.com/apollographql/apollo-client/blob/master/Upgrade.md

## Troubleshooting 

### Use of *.gql files

To use *gql|graphql files you need to add following dependency to your project:
```
  yarn add graphql-tag
  # alternative
  npm install graphql-tag
```

### Proxies

CORS errors are most often resolved with proxies.  If you see a Cross-Origin-Request error in your client side console look into setting up a proxy.  Check out https://github.com/nuxt-community/proxy-module for quick and straight forward setup.

###  ctx.req.session - req is undefined

This is just a placeholder.  You'll want to replace it with whatever storage mechanism you choose to store your token.
Here is an example using local storage : https://github.com/Akryum/vue-apollo/issues/144

## Contribute and wire up setup

Setup the required fields in .env file in root folder
```
// .env
HTTP_ENDPOINT=https://your-endpoint
WS_ENDPOINT=wss://your-endpoint
```

In `index.vue` the login process requires that the gql endpoint enables a mutation which returns a valid token:
```gql
mutation authenticateUser($email:String!,$password:String!){
    authenticateUser(email: $email, password: $password) {
        token
        id
    }
}
```

If your gql backend is prepared start running nuxt as follow
```
# npm install
# npm run dev
```
