# elm-parser-error-extra
This package allows to pretty print errors coming from `Parser.run`.

You will need a function which is specific to your output kind, be it the console, Html or Markdown. Those are presented below and not included in the package to keep the dependency list minimal.


## Colored console output
Depends on `wolfadex/elm-ansi`.
```elm
errorToString :
    String
    -> List (DeadEnd inner problem)
    -> String
errorToString src deadEnds =
    renderError
        { text = identity
        , colorContext = Ansi.Color.fontColor Ansi.Color.cyan
        , colorCaret = Ansi.Color.fontColor Ansi.Color.red
        , newline = "\n"
        , context = 3
        }
        Parser.Error.forParser -- or Parser.Error.forParserAdvanced
        src
        deadEnds
        |> String.concat
```


## Markdown
Depends on `dillonkearns/elm-markdown`.
```elm
errorToMarkdown :
    String
    -> List (DeadEnd inner problem)
    -> Block
errorToMarkdown src deadEnds =
    let
        color : String -> Inline -> Inline
        color value child =
            Block.HtmlInline
                (Block.HtmlElement "span"
                    [ { name = "style", value = "color:" ++ value } ]
                    [ Block.Paragraph [ child ] ]
                )
    in
    renderError
        { text = Block.CodeSpan
        , colorContext = color "cyan"
        , colorCaret = color "red"
        , newline = Block.HardLineBreak
        , context = 3
        }
        Parser.Error.forParser -- or Parser.Error.forParserAdvanced
        src
        deadEnds
        |> Block.Paragraph
```


## Html
Depends on `elm/html`.
```elm
errorToHtml :
    String
    -> List (DeadEnd inner Problem)
    -> Html msg
errorToHtml src deadEnds =
    let
        color : String -> Html msg -> Html msg
        color value child =
            Html.span [ Html.Attributes.style "color" value ] [ child ]
    in
    renderError
        { text = Html.text
        , colorContext = color "cyan"
        , colorCaret = color "red"
        , newline = Html.br [] []
        , context = 3
        }
        Parser.Error.forParser -- or Parser.Error.forParserAdvanced
        src
        deadEnds
        |> Html.pre []
```