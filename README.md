# clj-json-rpc

clj-json-rpc is a Clojure library that makes it easy to create web services
using the [JSON-RPC][1] protocol and [Ring][2]. 

The current version is 0.1. Just add this to your `project.clj`:

    [clj-json-rpc "0.1"]

With clj-json-rpc, you just define your JSON-RPC methods as normal Clojure 
functions, just substituting `defn-json-rpc` macro for `defn`. These methods
are in fact normal Clojure functions: for instance, you can call them at the
REPL or from other functions. The only way they're special is that they have
a `:json-rpc` key set to `true` in their metadata.

clj-json-rpc includes a Ring handler that you can use in your server, called
`process-json-rpc`. Typically you won't install it directly but instead invoke
it from your toplevel handler when some condition is satisfied, like this:

    (defn toplevel-handler [req]
      (if (= (:uri req) "/json-rpc") 
        (process-json-rpc req)
        (do-something-else req)))

The handler parses the request's body as JSON, extracts the necessary
fields as mandated by the JSON-RPC spec, calls the desired function
(as long as it has been defined using `defn-json-rpc`) and serializes
the output back to JSON. If the function throws an exception, it will
be reported as a JSON-RPC error.

Caveat: clj-json-rpc is currently very basic and limited in several ways.
One limitation is that you must define all your methods in the same namespace, 
or else the handler won't be able to locate them.

clj-json-rpc was originally part of the source of [Smyrna][3], a simple
concordancer for Polish, but was factored out since it is going to be used
in other projects as well. See that project for a usage example as well as
a sample of client code (in CoffeeScript).

## Example

    (use 'pl.danieljanus.jsonrpc)
    
    (defn-json-rpc example [x]
      (+ x 42)
    ;=> #'user/example
    
    ; simulate a request by directly calling the handler
    (process-json-rpc 
      {:body (java.io.ByteArrayInputStream. 
              (.getBytes "{\"method\":\"example\",\"params\":[2]}"))})
    ;=> {:status 200, 
         :headers {"Content-Type" "application/json; charset=utf-8"}, 
         :body "{\"result\":44}"}    

 [1]: http://json-rpc.org/
 [2]: http://github.com/mmcgrana/ring
 [3]: http://github.com/nathell/smyrna
