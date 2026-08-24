# Introduction

Le XSS consiste à exécuter du javascript arbitraire sur le navigateur d'une victime via divers vecteurs d'entrée.
```html
<input id="txt" type="text"/>
<input id="btn" type="button" value="click me :)"/>
<p id="res"></p>

<script>
	let txt = document.getElementById("txt");
	let btn = document.getElementById("btn");
	let res = document.getElementById("res");
	
	btn.addEventListener("click", () => {
		// XSS ici : n'importe quelle syntaxe de balise serait interprétée.
		res.innerHTML = txt.value;
	});
</script>
```

Si on tape `<img src=x onerror='alert()'/>`, une alerte apparaîtra car la balise sera interprétée.

- **XSS réfléchi** : manipulation d'entrées pour que le serveur injecte le code arbitraire (url, formulaire, ...).
- **XSS stocké** : stockage du code malveillant & interprétation pour quiconque visite la page infectée (pseudo, formulaire, ...).
- **XSS DOM** : manipulation d'entrées uniquement sur le client, sans intervention du serveur hôte.