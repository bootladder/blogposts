---
layout: post
title: SVG Fractals in Elm
---
<p>Using Elm, Ports, SVG to create interactive widgets.</p>
<p>
    <div id="elm-place">    </div>
    <script src="/assets/elmapp.js"></script>
    <script>
     var app = Elm.Main.init({
       node: document.getElementById('elm-place')
     });
    </script>
</p>

A good exercise.  
  
How to refactor this?
```
drawTreeThing : Int -> List (Svg msg)
drawTreeThing scale =
    let
        myLines =
            drawTreeOnPoints [ Point 0 0 ] (toFloat scale * 1 * pi / 16) 10

        reflectX =
            \line -> Line (Point line.p1.x (420 - line.p1.y) ) (Point line.p2.x (420 - line.p2.y)) line.depth

        reflectY =
            \line -> Line (Point (420 - line.p1.x) line.p1.y) (Point (420 - line.p2.x) line.p2.y) line.depth

        myLinesReflectedX =
            List.map reflectX myLines

        myLinesReflectedY =
            List.map reflectY myLines

        myLinesReflectedXY =
            List.map (reflectY << reflectX) myLines

        mySvgs =
            List.map line2Svg myLines

        mySvgsReflectedX =
            List.map line2Svg myLinesReflectedX

        mySvgsReflectedXY =
            List.map line2Svg myLinesReflectedXY

        mySvgsReflectedY =
            List.map line2Svg myLinesReflectedY
    in
    List.concat [ mySvgs, mySvgsReflectedX, mySvgsReflectedY, mySvgsReflectedXY ]

```

Remove some duplication with a function map  
```

drawTreeThing : Int -> List (Svg msg)
drawTreeThing scale =
    let
        myLines =
            drawTreeOnPoints [ Point 0 0 ] (toFloat scale * 1 * pi / 16) 10

        reflectX =
            \line -> Line (Point line.p1.x (420 - line.p1.y)) (Point line.p2.x (420 - line.p2.y)) line.depth

        reflectY =
            \line -> Line (Point (420 - line.p1.x) line.p1.y) (Point (420 - line.p2.x) line.p2.y) line.depth

        reflectXY =
            reflectX << reflectY

        myReflections =
            [ List.map reflectX, List.map reflectY, List.map reflectXY ]

        myLineSets =
            List.map (\fun -> fun <| myLines) myReflections

        myLinesConcat =
            List.concat myLineSets

        mySvgs =
            List.map line2Svg myLinesConcat
    in
    List.concat [ mySvgs ]
```
Remove excessive List.map and List.concat calls
```
drawTreeThing : Int -> List (Svg msg)
drawTreeThing scale =
    let
        myLines =
            drawTreeOnPoints [ Point 0 0 ] (toFloat scale * 1 * pi / 16) 10

        reflectX =
            \line -> Line (Point line.p1.x (420 - line.p1.y)) (Point line.p2.x (420 - line.p2.y)) line.depth

        reflectY =
            \line -> Line (Point (420 - line.p1.x) line.p1.y) (Point (420 - line.p2.x) line.p2.y) line.depth

        myReflections =
            [ reflectX, reflectY, reflectX << reflectY]

        myLineSets =
            List.map (\fun -> List.map fun myLines) myReflections
    in
    List.map line2Svg <| List.concat myLineSets
```
Now extract out of the reflectX function a Line Transform function  
```
drawTreeThing : Int -> List (Svg msg)
drawTreeThing scale =
    let
        myLines =
            drawTreeOnPoints [ Point 0 0 ] (toFloat scale * 1 * pi / 16) 10

        reflectPointX =
            \p -> Point p.x (420 - p.y)

        reflectPointY =
            \p -> Point (420 - p.x) p.y

        transformLine =
            \f1 f2 line -> Line (f1 line.p1) (f2 line.p2) line.depth

        reflectLineX =
            transformLine reflectPointX reflectPointX

        reflectLineY =
            transformLine reflectPointY reflectPointY

        myReflections =
            [ identity
            , reflectLineX
            , reflectLineY
            , reflectLineX << reflectLineY
            ]

        myLineSets =
            List.map (\fun -> List.map fun myLines) myReflections
    in
    List.map line2Svg <| List.concat myLineSets
```

Now here's the actual drawing of the tree.  
It was interesting to figure out what the types had to be
in order to make recursion work.  
```
drawTreeOnPoints :
    List Point
    -> Float
    -> Int
    -> List Line
drawTreeOnPoints points angle0 depth =
    case depth of
        0 ->
            []

        thisDepth ->
            let
                process =
                    \x -> draw2LinesOnPointReturning2Points x angle0 depth

                linesAndPoints : List ( List Line, List Point )
                linesAndPoints =
                    List.map process points

                linesOnly =
                    List.concat <| List.map Tuple.first linesAndPoints

                pointsOnly =
                    List.concat <| List.map Tuple.second linesAndPoints
            in
            linesOnly
                ++ drawTreeOnPoints pointsOnly (angle0 * 0.7) (thisDepth - 1)
```
