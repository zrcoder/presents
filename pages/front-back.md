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

[Source Code](https://github.com/zrcoder/ndor) | [Go WebAssembly](https://github.com/golang/go/wiki/WebAssembly)
