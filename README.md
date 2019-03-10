# aesophia_http

An HTTP interface to the Sophia compiler.It handles compiling
contracts and generating ACI data for contract.

HTTP interface uses port 3080, settable with application variable
aesophia_http.port.

Interface paths:

/aci - generate ACI format for contract in both JSON encoded and textual decoded forms. Tags 'code' and 'options'.

/compile - compile contract and return code in a JSON structure encoded to contract_bytearray.

/decode-data - Tags 'sophia-type' and 'data'.

/encode-calldata - Tags 'source', 'function' and 'arguments'.

/version - Return the sophia compiler version

/api - Return the API description.

```
contract="contract SimpleStorage =
  record state = { data : int }
  function init(value : int) : state = { data = value }
  function get() : int = state.data
  function set(value : int) = put(state{data = value})"
```

```
curl -H "Content-Type: application/json" -d "{\"code\":\"$contract\",\"options\":{}}" -X POST http://localhost:3080/aci

{"encoded_aci":{"contract":{"name":"SimpleStorage","type_defs":[{"name":"state","vars":[],"typedef":"{data : int}"}],"functions":[{"name":"init","arguments":[{"name":"value","type":"int"}],"type":"{data : int}","stateful":false},{"name":"get","arguments":[],"type":"int","stateful":false},{"name":"set","arguments":[{"name":"value","type":"int"}],"type":"()","stateful":false}]}},"interface":"contract SimpleStorage =\n  function get : () => int\n  function set : (int) => ()\n"}
```

```
curl -H "Content-Type: application/json" -d "{\"code\":\"$contract\",\"options\":{}}" -X POST http://localhost:3080/compile

{"bytecode":"cb_+QYYRgKgf6Gy7VnRXycsYSiFGAUHhMs+Oeg+RJvmPzCSAnxk8LT5BKX5AUmgOoWULXtHOgf10E7h2cFqXOqxa3kc6pKJYRpEw/nlugeDc2V0uMAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAoP//////////////////////////////////////////AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAC4YAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAP///////////////////////////////////////////jJoEnsSQdsAgNxJqQzA+rc5DsuLDKUV7ETxQp+ItyJgJS3g2dldLhgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA///////////////////////////////////////////uEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA+QKLoOIjHWzfyTkW3kyzqYV79lz0D8JW9KFJiz9+fJgMGZNEhGluaXS4wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACg//////////////////////////////////////////8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAALkBoAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEA//////////////////////////////////////////8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYD//////////////////////////////////////////wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAuQFEYgAAj2IAAMKRgICAUX9J7EkHbAIDcSakMwPq3OQ7LiwylFexE8UKfiLciYCUtxRiAAE5V1CAgFF/4iMdbN/JORbeTLOphXv2XPQPwlb0oUmLP358mAwZk0QUYgAA0VdQgFF/OoWULXtHOgf10E7h2cFqXOqxa3kc6pKJYRpEw/nlugcUYgABG1dQYAEZUQBbYAAZWWAgAZCBUmAgkANgAFmQgVKBUllgIAGQgVJgIJADYAOBUpBZYABRWVJgAFJgAPNbYACAUmAA81tgAFFRkFZbYCABUVGQUIOSUICRUFCAWZCBUllgIAGQgVJgIJADYAAZWWAgAZCBUmAgkANgAFmQgVKBUllgIAGQgVJgIJADYAOBUoFSkFCQVltgIAFRUVlQgJFQUGAAUYFZkIFSkFBgAFJZkFCQVltQUFlQUGIAAMpWhTIuMC4wymxiIQ=="}
```

```
curl -H "Content-Type: application/json" -d "{\"function\":\"set\",\"arguments\":[\"42\"],\"source\":\"$contract\"}" -X POST http://localhost:3080/encode-calldata

{"calldata":"cb_AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACA6hZQte0c6B/XQTuHZwWpc6rFreRzqkolhGkTD+eW6BwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACoA4Uun"}
```

```
curl -H "Content-Type: application/json" -d "{\"data\":\"cb_AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACr8s/aY\",\"sophia-type\":\"int\"}" -X POST http://localhost:3080/decode-data

{"data":{"type":"word","value":42}}
```

```
curl http://localhost:3080/api

{"api":{"swagger":"2.0","info":{"description":"This is the [Aeternity](https://www.aeternity.com/) compiler API.","version":"2.0.0","title":"Aeternity node","termsOfService":"https://www.aeternity.com/terms/","contact":{"email":"apiteam@aeternity.com"}},"basePath":"/","schemes":["http"],"paths":{"/aci":{"post":{"description":"Generate an Aeternity Contract Interface (ACI) for contract","operationId":"GenerateACI","consumes":["application/json"],"produces":["application/json"],"parameters":[{"in":"body","name":"body","description":"contract code","required":true,"schema":{"$ref":"#/definitions/Contract"}}],"responses":{"200":{"description":"ACI for contract","schema":{"$ref":"#/definitions/ACI"}},"403":{"description":"Invalid input","schema":{"$ref":"#/definitions/Error"}}}}},"/compile":{"post":{"description":"Compile a sophia contract from source and return byte code","operationId":"CompileContract","consumes":["application/json"],"produces":["application/json"],"parameters":[{"in":"body","name":"body","description":"contract code","required":true,"schema":{"$ref":"#/definitions/Contract"}}],"responses":{"200":{"description":"Byte code response","schema":{"$ref":"#/definitions/ByteCode"}},"403":{"description":"Invalid contract","schema":{"$ref":"#/definitions/Error"}}}}},"/decode-data":{"post":{"description":"Decode data as retuned by a contract call. - Legacy decoding","operationId":"DecodeData","consumes":["application/json"],"produces":["application/json"],"parameters":[{"in":"body","name":"body","description":"Binary data in Sophia ABI format","required":true,"schema":{"$ref":"#/definitions/SophiaBinaryData"}}],"responses":{"200":{"description":"Json encoded data","schema":{"$ref":"#/definitions/SophiaJsonData"}},"400":{"description":"Invalid data","schema":{"$ref":"#/definitions/Error"}}}}},"/encode-calldata":{"post":{"description":"Encode Sophia function call according to sophia ABI.","operationId":"EncodeCalldata","consumes":["application/json"],"produces":["application/json"],"parameters":[{"in":"body","name":"body","description":"Sophia function call - contract code + function name + arguments","required":true,"schema":{"$ref":"#/definitions/FunctionCallInput"}}],"responses":{"200":{"description":"Binary encoded calldata","schema":{"$ref":"#/definitions/Calldata"}},"403":{"description":"Invalid contract","schema":{"$ref":"#/definitions/Error"}}}}},"/version":{"get":{"description":"Get the version of the underlying Sophia compiler version","operationId":"Version","produces":["application/json"],"parameters":[],"responses":{"200":{"description":"Sophia compiler version","schema":{"$ref":"#/definitions/CompilerVersion"}},"403":{"description":"Error","schema":{"$ref":"#/definitions/Error"}}}}},"/api":{"get":{"description":"Get the Api description","operationId":"Api","produces":["application/json"],"parameters":[],"responses":{"200":{"description":"API description","schema":{"$ref":"#/definitions/API"}},"403":{"description":"Error","schema":{"$ref":"#/definitions/Error"}}}}}},"definitions":{"Contract":{"type":"object","required":["code","options"],"properties":{"code":{"type":"string"},"options":{"$ref":"#/definitions/CompileOpts"}},"example":{"code":"code","options":{"src_file":"src_file","file_system":"{}"}}},"CompileOpts":{"type":"object","properties":{"file_system":{"type":"object","description":"An explicit file system, mapping file names to file content","properties":{}},"src_file":{"type":"string","description":"Name of contract source file - only used in error messages"}},"example":{"src_file":"src_file","file_system":"{}"}},"CompilerVersion":{"type":"object","required":["version"],"properties":{"version":{"type":"string","description":"Sophia compiler version"}},"example":{"version":"version"}},"Error":{"type":"object","required":["reason"],"properties":{"reason":{"type":"string"}}},"ACI":{"type":"object","required":["encoded_aci","interface"],"properties":{"encoded_aci":{"type":"object","properties":{}},"interface":{"type":"string"}},"example":{"interface":"interface","encoded_aci":"{}"}},"API":{"type":"object","properties":{"api":{"type":"object","description":"Swagger API description","properties":{}}},"example":{"api":"{}"}},"ByteCode":{"type":"object","required":["bytecode"],"properties":{"bytecode":{"$ref":"#/definitions/EncodedByteArray"}},"example":{"bytecode":{}}},"SophiaBinaryData":{"type":"object","required":["data","sophia-type"],"properties":{"sophia-type":{"type":"string"},"data":{"type":"string"}},"example":{"data":"data","sophia-type":"sophia-type"}},"SophiaJsonData":{"type":"object","required":["data"],"properties":{"data":{"type":"object","properties":{}}},"example":{"data":"{}"}},"FunctionCallInput":{"type":"object","required":["arguments","function","source"],"properties":{"source":{"type":"string","description":"(Possibly partial) Sophia contract code"},"function":{"type":"string","description":"Name of function to call"},"arguments":{"type":"array","description":"Array of function call arguments","items":{"type":"string"}}},"example":{"function":"function","arguments":["arguments","arguments"],"source":"source"}},"Calldata":{"type":"object","required":["calldata"],"properties":{"calldata":{"$ref":"#/definitions/EncodedByteArray"}},"example":{"calldata":{}}},"EncodedByteArray":{"type":"string","description":"Prefixed (cb_) Base64Check encoded byte array"}}}}
```

```
curl http://localhost:3080/version

{"version":"2.0.0"}
```