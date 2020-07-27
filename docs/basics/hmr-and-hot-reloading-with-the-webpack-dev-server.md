# HMR and Hot Reloading with the webpack-dev-server

The webpack-dev-server provides:

1. Speedy compilation of client side assets
2. Optional HMR which means that the page will reload automatically when after
   compilation completes. Note, some developers do not like this, as you'll
   abruptly lose any tweaks within the Chrome development tools.
3. Optional hot-reloading. The older react-hot-loader has been deprecated in 
   favor of [fast-refresh](https://reactnative.dev/docs/fast-refresh).
   For use with webpack, see [react-refresh-webpack-plugin](https://github.com/pmmmwh/react-refresh-webpack-plugin).

If you are ***not*** using server-side rendering (***not*** using `prerender: true`),
then you can follow all the regular docs for using the `bin/webpack-dev-server` 
during development.


# Server Side Rendering with the Default rails/webpacker bin/webpack-dev-server

If you are using server-side rendering, then you have a couple options. The
recommended technique is to have a different webpack configuration for server
rendering.  




## If you use the same Webpack setup for your server and client bundles 
If you do use the webpack-dev-server for prerendering, be sure to set the
`config/initializers/react_on_rails.rb` setting of 

```
  config.same_bundle_for_client_and_server = true
```

`dev_server.hmr` maps to [devServer.hot](https://webpack.js.org/configuration/dev-server/#devserverhot).
This must be false if you're using the webpack-dev-server for client and server bundles.
 
`dev_server.inline` maps to [devServer.inline](https://webpack.js.org/configuration/dev-server/#devserverinline).
This must also be false.

If you don't configure these two to false, you'll see errors like:

* "ReferenceError: window is not defined" (if hmr is true)
* "TypeError: Cannot read property 'prototype' of undefined" (if inline is true)

# HMR using react-refresh-webpack-plugin
If you use the webpack-dev-server and want to enable HMR follow next steps:

1. In `config/webpacker.yml` set **hmr** and **inline** server properties to true 
```
   dev_server:
    https: false
    host: localhost
    port: 3035
    public: localhost:3035
    hmr: true
    # Inline should be set to true if using HMR
    inline: true
```
2. Add react refresh packages:
` npm install @pmmmwh/react-refresh-webpack-plugin react-refresh --development` or ` yarn add @pmmmwh/react-refresh-webpack-plugin react-refresh -D`

3. In development environment `config/webpack/development.js` add react-refresh-webpack-plugin in plugins array and react-refresh plugin for babel loader.

```
  //plugins
  environment.plugins.append(
     'ReactRefreshWebpackPlugin',
      isDevelopment && new ReactRefreshWebpackPlugin()
  );

  //loaders
  const babelLoader = environment.loaders.get('babel');
  babelLoader.use[0].options.plugins = [].filter(Boolean);
  isDevelopment &&  babelLoader.use[0].options.plugins.push(require.resolve('react-refresh/babel'));

```
Thats it :).
Now Browser should reflect changes in your .js code.

On sockjs error in browser console `GET http://localhost:[port]/sockjs-node/info?t=[xxxxxxxxxx] 404 (Not Found)` you have to add overlay false option to `ReactRefreshWebpackPlugin`
 ```
 new ReactRefreshWebpackPlugin({
   overlay: false
 })
```

If you have troubles with rspec tests you could wrap plugins in conditional `if(process.env.RAILS_ENV === 'development')`. 
If by some reason plugin doesnt work you could revert changes and left only devServer hmr/inline to true affecting only css files.

These plugins are working and tested with babel 7, webpacker 5, bootstrap 4, jest 26, core-js 3, node 12.10.0, react-refresh-webpack-plugin 0.0.3, react-refresh 0.8.3 configuration
