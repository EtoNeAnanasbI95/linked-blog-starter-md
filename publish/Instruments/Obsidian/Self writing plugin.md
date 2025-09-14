# Начало:

> [!INFO] База
> Я использовал [официальную документацию документацию](https://docs.obsidian.md/Plugins/Getting+started/Build+a+plugin#What%20you'll%20learn) к созданию плагинов для обсидана, так что в случае чего просто проверяйте там
* Если в кратце, то там мы просто создали хранилище, сделали папку в .obsidian, назвали её plugins, и склонировали туда семпл плагина
* Начальная структура проекта
```txt
.
├── AGENTS.md
├── esbuild.config.mjs
├── LICENSE
├── main.js
├── main.ts
├── manifest.json
├── package-lock.json
├── package.json
├── README.md
├── styles.css
├── tsconfig.json
├── version-bump.mjs
└── versions.json
```


Можно вызывать модалки через колл беки к функциям:
```ts
		this.addCommand({
			id: 'open-sample-modal-simple',
			name: 'Open sample modal (simple)',
			callback: () => {
				new SampleModal(this.app).open();
			}
		});
```