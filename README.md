# How make [remark42 commenting engine](https://github.com/umputun/remark42) work with Single Page Apps (SPAs)

Originally tested on [Nuxt.js](https://nuxtjs.org/), but it should be applicable to all SPAs.

- Add the following JavaScript to your `index.html`, which in this case, it is identical to `<script defer src="$HOST/web/embed.js"></script>`

```js
;(function () {
  var host = // Your remark42 host
  var components = ['embed'] // Your choice of remark42 components

  ;(function(c) {
    for (let i = 0; i < c.length; i++) {
      const d = document
      const s = d.createElement('script')
      s.src = remark_config.host + '/web/' + c[i] + '.js'
      s.defer = true
      ;(d.head || d.body).appendChild(s)
    }
  })(components)
})
```

- Created `remark42Instance` when the `div` containing remark42 has appear, usually at `mounted` or `componentDidMount` of the SPA lifecycle. Destroy the previous instance first, if neccessary.

```ts
  mounted() {
    if (window.REMARK42) {
      if (window.REMARK42.destroy) {
        window.REMARK42.destroy()
      }

      window.REMARK42.createInstance({
        node: this.$refs.remark42 as HTMLElement,
        ...remark42_config  // See <https://github.com/patarapolw/remark42#setup-on-your-website>
      })
    }
  }
```

- Ensure that this is called every time route changes

```ts
  @Watch('$route.path')
  onRouteChange() {
    if (window.REMARK42) {
      if (window.REMARK42.destroy) {
        window.REMARK42.destroy()
      }

      window.REMARK42.createInstance({
        node: this.$refs.remark42 as HTMLElement,
        host: config.host,
        site_id: config.siteId
      })
    }
  }
```

- And, destroyed before routeLeave

```ts
  beforeRouteLeave() {
    if (process.client) {
      if (window.REMARK42 && window.REMARK42.destroy) {
        window.REMARK42.destroy()
      }
    }
  }
```
