# 前端 ⇠⇢ 后端

<iframe src='https://ndor.netlify.app' width='100%' height='90%' />

---

## 用 webassymbly 统一前后端

```go {1,7}
func (idx *index) Render() app.UI {
	return app.Div().Style("overflow", "hidden").Body(
		// ...
		app.Button().
			ID("run-button").
			Class("run-button").
			OnClick(goButtonAction)
			Text("GO"),
	)
}
```

```go
func goButtonAction(ctx app.Context, e app.Event) {
	root := app.Window()
	code := root.Get("GetCode").Invoke().String()
	imgData, err := pkg.Run(width, height, code)
	// ...
	root.Get(pictureAreaID).Set("src", imgData)
}
```

[Source Code](https://github.com/zrcoder/ndor) | [Go WebAssembly](https://go.dev/wiki/WebAssembly)

---

## Amisgo

组件丰富的纯 Go 语言 web 框架，前端为主，也非常容易集成后端，写前后端合一的 web 应用。

![dev-toys-by-amisgo](https://raw.githubusercontent.com/zrcoder/amisgo-examples/main/dtoy.png)

[amisgo](https://github.com/zrcoder/amisgo) | [amisgo-examples](https://github.com/zrcoder/amisgo-examples)
