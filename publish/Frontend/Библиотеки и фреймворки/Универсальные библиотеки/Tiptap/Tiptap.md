[Tiptap](https://tiptap.dev/docs/editor/getting-started/overview) — это мощная библиотека для создания WYSIWYG (What You See Is What You Get) редакторов в веб-приложениях. Она построена на основе [[ProseMirror]] и предоставляет гибкий и расширяемый API, что позволяет разработчикам легко настраивать редакторы под свои нужды.

### Основные особенности Tiptap:

-  **Модульность**: Tiptap позволяет добавлять и удалять функции редактора с помощью [расширений](https://tiptap.dev/docs/editor/extensions/overview).
-  **Поддержка различных форматов**: Поддерживает текст, изображения, таблицы и другие элементы.
-  **Совместимость**: Легко интегрируется с популярными фреймворками, такими как Vue и React.
-  **Настраиваемый интерфейс**: Можно настраивать внешний вид и поведение редактора.

Tiptap идеально подходит для создания текстовых редакторов, блогов, систем управления контентом и других приложений, где требуется редактирование текста.
# Аналоги:
* React Quill
# Важно:
* Пока на ней писал, надо было сделать текстовый редактор как в врод, типа вот такой
![[Pasted image 20250529144628.png]]
* И короче заебался я с шрифтами, чтоб динамически обновлялись, с их размерами, и с их цветом
* Так же чтоб вставить фотку в редактор, есть платное расширение, но его нельзя использовать коммерции, потому пришлось делать своё
### font-size.ts
```ts
import { Extension } from '@tiptap/core'

export interface FontSizeOptions {
  types: string[]
}

declare module '@tiptap/core' {
  interface Commands<ReturnType> {
    fontSize: {
      setFontSize: (fontSize: string) => ReturnType
    }
  }
}

export const FontSize = Extension.create<FontSizeOptions>({
  name: 'fontSize',

  addOptions() {
    return {
      types: ['textStyle'],
    }
  },

  addGlobalAttributes() {
    return [
      {
        types: this.options.types,
        attributes: {
          fontSize: {
            default: null,
            parseHTML: (element) => element.style.fontSize,
            renderHTML: (attributes) => {
              if (!attributes.fontSize) {
                return {}
              }

              return {
                style: `font-size: ${attributes.fontSize}`,
              }
            },
          },
        },
      },
    ]
  },

  addCommands() {
    return {
      setFontSize:
        (fontSize: string) =>
        ({ chain }) => {
          return chain().setMark('textStyle', { fontSize }).run()
        },
    }
  },
})

```
### images-paste.ts
```ts
import { Image as Images } from '@tiptap/extension-image'
import { Plugin } from 'prosemirror-state'

const Image = Images.extend({
  addAttributes() {
    return {
      ...this.parent?.(),
      selected: {
        default: false,
        renderHTML: (attributes) => {
          return attributes.selected ? { 'data-selected': 'true' } : {}
        },
      },
    }
  },

  renderHTML({ node }) {
    const attrs = {
      ...node.attrs,
      'data-selected': node.attrs.selected ? 'true' : 'false',
    }
    return ['img', attrs]
  },

  addProseMirrorPlugins() {
    return [
      new Plugin({
        props: {
          handleDOMEvents: {
            click: (view, event) => {
              const { state, dispatch } = view
              const { doc, tr } = state
              const target = event.target as HTMLElement

              if (target.tagName.toLowerCase() === 'img') {
                const pos = view.posAtDOM(target, 0)
                if (pos !== undefined) {
                  let newTr = tr
                  doc.descendants((node, pos) => {
                    if (node.type.name === 'image' && node.attrs.selected) {
                      newTr = newTr.setNodeMarkup(pos, undefined, {
                        ...node.attrs,
                        selected: false,
                      })
                    }
                  })

                  newTr = newTr.setNodeMarkup(pos, undefined, {
                    ...doc.nodeAt(pos)?.attrs,
                    selected: true,
                  })

                  dispatch(newTr)
                  return true
                }
              } else {
                let newTr = tr
                doc.descendants((node, pos) => {
                  if (node.type.name === 'image' && node.attrs.selected) {
                    newTr = newTr.setNodeMarkup(pos, undefined, {
                      ...node.attrs,
                      selected: false,
                    })
                  }
                })
                dispatch(newTr)
              }
              return false
            },
            keydown: (view) => {
              const { state, dispatch } = view
              const { doc, tr } = state

              // Снимаем выделение со всех изображений при нажатии любой клавиши
              let newTr = tr
              doc.descendants((node, pos) => {
                if (node.type.name === 'image' && node.attrs.selected) {
                  newTr = newTr.setNodeMarkup(pos, undefined, {
                    ...node.attrs,
                    selected: false,
                  })
                }
              })
              dispatch(newTr)
              return false
            },
            drop(view, event) {
              const hasFiles =
                event.dataTransfer &&
                event.dataTransfer.files &&
                event.dataTransfer.files.length

              if (!hasFiles) return

              const images = Array.from(event.dataTransfer.files).filter(
                (file) => /image/i.test(file.type),
              )

              if (images.length === 0) return

              event.preventDefault()

              const { schema } = view.state
              const coordinates = view.posAtCoords({
                left: event.clientX,
                top: event.clientY,
              })

              images.forEach((image) => {
                const reader = new FileReader()

                reader.onload = (readerEvent) => {
                  const node = schema.nodes.image.create({
                    src: readerEvent.target?.result,
                    selected: false,
                  })
                  const transaction = view.state.tr.insert(
                    coordinates?.pos as number,
                    node,
                  )
                  view.dispatch(transaction)
                }
                reader.readAsDataURL(image)
              })
            },
            paste(view, event) {
              const hasFiles =
                event.clipboardData &&
                event.clipboardData.files &&
                event.clipboardData.files.length

              if (!hasFiles) return

              const images = Array.from(event.clipboardData.files).filter(
                (file) => /image/i.test(file.type),
              )

              if (images.length === 0) return

              event.preventDefault()

              const { schema } = view.state

              images.forEach((image) => {
                const reader = new FileReader()

                reader.onload = (readerEvent) => {
                  const node = schema.nodes.image.create({
                    src: readerEvent.target?.result,
                    selected: false,
                  })
                  const transaction = view.state.tr.replaceSelectionWith(node)
                  view.dispatch(transaction)
                }
                reader.readAsDataURL(image)
              })
            },
          },
        },
      }),
    ]
  },
})

export { Image }

```
* При этом, чтобы динамически получать текущий шрифт, его размер и цвет я написал вот это:
  ```tsx
  const [currentFont, setCurrentFont] = useState<string>(
    DEFAULT_FONT_FACE_OPTIONS[0].content,
  )
  const [currentSize, setCurrentSize] = useState<string>(FONT_SIZES[4].content)
  const [currentColor, setCurrentColor] = useState<string>('#000000')

// eslint-disable-next-line react-hooks/rules-of-hooks
  const updateAttributes = useCallback(() => {
    const attrs = editor.getAttributes('textStyle')
    setCurrentFont(attrs.fontFamily || DEFAULT_FONT_FACE_OPTIONS[0].content)
    setCurrentSize(attrs.fontSize || FONT_SIZES[4].content)
    setCurrentColor(attrs.color || '#000000')
  }, [editor])

  // eslint-disable-next-line react-hooks/rules-of-hooks
  useEffect(() => {
    if (!editor) return

    editor.on('selectionUpdate', updateAttributes)
    editor.on('transaction', updateAttributes)

    updateAttributes()

    return () => {
      editor.off('selectionUpdate', updateAttributes)
      editor.off('transaction', updateAttributes)
    }
  }, [editor, updateAttributes])
```