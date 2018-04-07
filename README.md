AppBase
--------------

The AppBase library provides a basic framework for building applications from
a set of plugins. AppBase manages the plugin life-cycle and ensures that all
plugins are configured, initialized, started, and shutdown in the proper order.

## Key Features

- Dynamically Specify Plugins to Load
- Automatically Load Dependent Plugins in Order
- Plugins can specify commandline arguments and configuration file options
- Program gracefully exits from SIGINT and SIGTERM
- Minimal Dependencies (Boost 1.60, c++14)

## Defining a Plugin

A simple example of a 2-plugin application can be found in the /examples directory. Each plugin has
a simple life cycle:

1. Initialize - parse configuration file options
2. Startup - start executing, using configuration file options
3. Shutdown - stop everything and free all resources

All plugins complete the Initialize step before any plugin enters the Startup step. Any dependent plugin specified
by `APPBASE_PLUGIN_REQUIRES` will be Initialized or Started prior to the plugin being Initialized or Started. 

This example can be used like follows:

```
./examples/appbase_example --plugin net_plugin
initialize chain plugin
initialize net plugin
starting chain plugin
starting net plugin
^C
shutdown net plugin
shutdown chain plugin
exited cleanly
```

### Boost ASIO 

AppBase maintains a singleton `application` instance which can be accessed via `appbase::app()`.  This 
application owns a `boost::asio::io_service` which starts running when `appbase::exec()` is called. If 
a plugin needs to perform IO or other asynchronous operations then it should dispatch it via 
`app().get_io_service().post( lambda )`.  

Because the app calls `io_service::run()` from within `application::exec()` all asynchronous operations
posted to the io_service should be run in the same thread.  

## Graceful Exit 

To trigger a graceful exit call `appbase::app().quit()` or send SIGTERM or SIGINT to the process.

## Dependencies 

1. c++14 or newer  (clang or g++)
2. Boost 1.60 or newer compiled with C++14 support

To compile boost with c++14 use:

```
./b2 ...  cxxflags="-std=c++0x -stdlib=libc++" linkflags="-stdlib=libc++" ...
```

