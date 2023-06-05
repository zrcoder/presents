# 前端 ⇠⇢ 后端

用 webassymbly 联通前后端

---
src: ndor.md
---

---

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
	alert := root.Get("getNiudourAlert").Invoke()
	code := root.Get("GetCode").Invoke().String()
	alert.Call("toastPainting")
	imgData, err := pkg.Run(width, height, code)
	// ...
	alert.Call("closePaintToast")
	root.Get(pictureAreaID).Set("src", imgData)
}
```

