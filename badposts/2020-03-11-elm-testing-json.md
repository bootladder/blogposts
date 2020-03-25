---
layout: post
title:  "Elm JSON Testing:  Insert JSON into Elm Test"
---

I've been TDDing recently and I have an Elm app which needs to receive JSON via HTTP request.  Well of course I'm not going to actually make an HTTP request for the purpose of testing my code!  

https://becoming-functional.com/testing-json-decoders-in-elm-39f613a98895
  
This is a good article, except I don't want my test input in my elm test.  My idea is that I have a `testresponse.json` and I can use that file for testing both my frontend and backend.  

https://stackoverflow.com/questions/48378601/load-external-data-into-elm-test-suite  
  
That's the problem, and the solution I'm going for is the template.  But how do I do the template?  
  
What's the easiest way to do that?  

Here's what I got.  Do you have anything better?  
```
#!/bin/bash
set -e

printf "$(cat tests/JsonDecodeTest.elm.tmpl)" "$(cat example.json)" > tests/JsonDecodeTest.elm
elm-test

elm make src/Main.elm --output main.js

```
```
module JsonDecodeTest exposing (..)

import Main exposing (..)
import Json.Decode

import Expect exposing (Expectation)
import Test exposing (..)

testJson : String
testJson =
    """
    %s
    """

suite : Test
suite =
    describe "Backend JSON Response Decoder"
        [ test "doesn't fail" <|
            \_ ->
                let
                    decodedOutput =
                        Json.Decode.decodeString
                            backendResponseDecoder testJson
                in
                    Expect.equal testJson "b"
        ]

```
  
It works but 1 problem is that the elm file ends with .tmpl
taking away syntax highlighting.
