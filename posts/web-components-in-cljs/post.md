I read about web components years go, but never really had the time to try them out. The promise of web components is attractive, you get framework independence, native browser performance, strict encapsulation all for free. When I finally got around to trying them out it was born out of some frustration with my current approach for client development with Clojurescript, re-frame, react and then a ton of libraries that solve all the nitty gritty details that need addressing in a large client application. The setup works but it isn't easy.

I've written down why I do not think it is easy (or simple) at length [here](https://avanelsas.github.io/foggy-glasses/on-client-state-complexity/).

My initial naive view of web components was that they might reduce a lot of the complexity of creating and managing components, reduce some of the inner state you are always forced to handle (e.g. an input that does not validate), and that bundle size would reduce heavily.

But there are downsides as well, some obvious, others a lot more subtle. The obvious one is that web components may be powerful, but the API is not simple. It requires quite a bit of mental gymnastics to get your head around how web components work and are implemented.

A subtle downside is that web components are truly low level. There is a reason all these different rendering frameworks exist. Each of them tries to solve problems web components do not even begin to address. The most complex one perhaps is that there is no simple way to determine if something needs to be re-rendered on screen. A view with 10 web components might re-render everything, possibly multiple times, if you only touch one small part or attribute.

There are other potential downsides as well, but the rendering challenge is probably one of the most complex ones. Each can be addressed, but you need to be aware they exist in the first place.

## What is a web component

```html live
<style>
    x-bento-grid { width: 100%; }
    .cell { display:flex; align-items:center; justify-content:center; height:100%;
            box-sizing:border-box; border-radius:12px; padding:1.5rem;
            font-weight:600; font-size:0.875rem; }
    .cell-1 { background:#dbeafe; color:#1e40af; }
    .cell-2 { background:#fce7f3; color:#9d174d; }
    .cell-3 { background:#d1fae5; color:#065f46; }
    .cell-4 { background:#fef3c7; color:#92400e; }
    .cell-5 { background:#e0e7ff; color:#3730a3; }
    .cell-6 { background:#ede9fe; color:#5b21b6; }
    @media (prefers-color-scheme: dark) {
      .cell-1 { background:#1e3a5f; color:#93c5fd; }
      .cell-2 { background:#4a1942; color:#f9a8d4; }
      .cell-3 { background:#064e3b; color:#6ee7b7; }
      .cell-4 { background:#451a03; color:#fcd34d; }
      .cell-5 { background:#312e81; color:#a5b4fc; }
      .cell-6 { background:#3b0764; color:#c4b5fd; }
    }
  </style>

<x-bento-grid columns="4" gap="md" row-height="120px">
    <x-bento-item col-span="2" row-span="2" style="grid-area: span 2 / span 2;">
      <div class="cell cell-1" style="height:100%">Featured</div>
    </x-bento-item>
    <x-bento-item row-span="2" style="grid-area: span 2 / span 1;">
      <div class="cell cell-3" style="height:100%">Tall</div>
    </x-bento-item>
    <x-bento-item style="grid-area: span 1 / span 1;">
      <div class="cell cell-2">Small A</div>
    </x-bento-item>
    <x-bento-item style="grid-area: span 1 / span 1;">
      <div class="cell cell-4">Small B</div>
    </x-bento-item>
    <x-bento-item col-span="3" style="grid-area: span 1 / span 3;">
      <div class="cell cell-5">Wide banner</div>
    </x-bento-item>
    <x-bento-item style="grid-area: span 1 / span 1;">
      <div class="cell cell-6">End</div>
    </x-bento-item>
  </x-bento-grid>
```

A web component is a custom HTML element. Once you create and register it, it acts like a native html element. You can then use it like how you would use `<button>` in plain html. In the example above I've declared `x-bento-item` and `x-bento-grid` web components to create the grid displayed.

There are 3 browser standards that a make up a web component:

- Custom Elements connect a tag name to a class, and the browser will call that class when it needs to be rendered
- Shadow DOM gives the element a private tree with styles and structure that will not mix with the page.
- Templates supply the internal structure and will show the web component content at fixed positions in that structure

## What does an implementation have to cover

If you want to create your own web component you need to implement a number of things yourself:

- Identity: a name with a hyphen.
- Inputs: attributes for HTML and properties for JavaScript. They share one source of truth.
- Rendering: one function that makes the display match the current inputs.
- Lifecycle: work on entry, on each change, and clean-up on exit. Each step must be safe to repeat.
- Encapsulation: a shadow DOM that keeps the internal structure and styles private.
- Content: slots that place the user's content inside the element.
- Outputs: events that tell the page about user actions and reach it across the shadow boundary.
- Styling: custom properties and named parts, so a page can theme the element from outside.
- Accessibility: keyboard support, roles and labels that a screen reader can read.
- Forms: when the element is a control, a value, validation and reset, so it works in a form like a native input.

## Writing a Web Component in CLJS

Please note that **this explanation is directly related to how the [BareDOM](https://github.com/avanelsas/baredom) library** is written. It is therefore opinionated. The basic gist is the same for any web component, but I've spent quite some time on creating a render pipeline that is readable and efficient for the web components in the library.

Web components can be as complex as you want them to be. And fun too. Here is an example of a web component called x-gaussian-blur. I like it because it is both colourful, animates, and is mostly useful if you want to let your design be more organic and animating. I find it mesmerizing to look at it.

```html live height=250
<x-gaussian-blur data-animation="float" role="presentation" aria-hidden="true"></x-gaussian-blur>
```

## Let's start writing a Web Component in Clojurescript.

I will show how to write a relatively simple one, called x-icon. It is basically a themeable wrapper around an svg. Here is an x-icon that shows a small heart icon:

```html live
<x-icon size="md" color="primary" label="Save">
  <svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 21s-7-4.35-9.5-9.5C.7 7.6 3.2 3.5 7 3.5c2 0 3.5 1 5 3 1.5-2 3-3 5-3 3.8 0 6.3 4.1 4.5 8C19 16.65 12 21 12 21z"></path></svg>
</x-icon>
```

### Step 1: start with data

Every web component will use some form of data. Think of it as the model that the web component will read. In a `model.cljs` file for x-icon we can write the following attributes for the component.

```html
(def tag-name "x-icon")

(def attr-size  "size")
(def attr-color "color")
(def attr-label "label")

;; declare the attributes to be observed by the web component
(def observed-attributes #js [attr-size attr-color attr-label])
```

In BareDOM every model has 5 definitions that are used, `tag-name`, `observed-attributes`, `property-api`, `event-schema` and `method-api`.

Then declare the public API for the web component.

```html
(def property-api
   {:size  {:type 'string :reflects-attribute attr-size :default ""}
    :color {:type 'string :reflects-attribute attr-color :default ""}
    :label {:type 'string :reflects-attribute attr-label :default ""}})

 (def event-schema {}) ;; can be empty
 (def method-api {})   ;; can be empty
 ```

 As every attribute needs a property that reflects it, it is important to standardise how you want to deal with that. In our case we will always use camel-case names for property names. And equally important, you need to ensure that a property only holds a clean value. This means you need to write a small parser for each attribute to ensure that the web component does not receive bad input. In BareDOM a choice is made that bad input does not return an error but will instead return a default value.

 An example is the `size` property. It accepts both `"sm"`/`"md"`/`"lg"`/`"xl"`, but it can also handle `"48"`. A parser for that might look like this:

 ```html
 (defn parse-size
    [s]
    (or (parse-size-token  s) ;; handles `"sm"`/`"md"`/`"lg"`/`"xl"` parsing
        (parse-size-pixels s) ;; handles `"48"` parsing
        default-size-map))    ;; else a default
  ```

  The next step is to collect and normalise any raw input into the view model the component will display. I try to keep code in the model file pure. Normalisation for x-icon is done as follows:

  ```html
  (defn normalize
    [{:keys [size-raw color-raw label-raw label-present?]}]
    (let [size  (parse-size  size-raw)
          color (parse-color color-raw)
          label (if (and label-present? (string? label-raw)) label-raw "")]
      {:size-css  (:css size)
       :color-css (color-css-value color)
       :label     label
       ;; boolean is used here to coerse a potential nil value to a boolean
       :labelled? (boolean (and label-present? (not= "" label)))}))
   ```

 There are more useful functions and definitions in the model file for x-icon, you can see the code here [`model.cljs`](https://github.com/avanelsas/baredom/blob/main/src/baredom/components/x_icon/model.cljs).

   Onto the x-icon.cljs file, where the DOM layer is created. This is where the side effects will live. First, every component will likely need a few definitions to access instance fields, so we declare them at the top of the file:

```html
   (def ^:private k-initialized? "__xIconInitialized")
   (def ^:private k-model        "__xIconModel")

   (def ^:private css-var-size  "--x-icon-size")
   (def ^:private css-var-color "--x-icon-color")

   (def ^:private attr-role        "role")
   (def ^:private attr-aria-label  "aria-label")
   (def ^:private attr-aria-hidden "aria-hidden")

   (def ^:private attr-part "part")
   (def ^:private part-box  "box")

   (def ^:private role-img         "img")
   (def ^:private aria-hidden-true "true")

   (def ^:private style-text
     (str
      ":host{"
      "display:inline-flex;"
      "align-items:center;"
      "justify-content:center;"
      "line-height:0;"
      "vertical-align:middle;"
      "color:var(" css-var-color ",currentColor);"
      css-var-size ":20px;}"
      du/hidden-rule

      "[part=box]{"
      "display:inline-flex;"
      "align-items:center;"
      "justify-content:center;"
      "width:var(" css-var-size ");"
      "height:var(" css-var-size ");}"

      "::slotted(svg){"
      "display:block;"
      "width:100%;"
      "height:100%;}"

      "::slotted(svg[fill=\"currentColor\"]),"
      "::slotted(svg[stroke=\"currentColor\"]){color:inherit;}"))
```

An instance field can keep a shadow root, DOM node handles or the cached model. It will never keep UI state. The next step is a rather messy one imo, you need to write the styles the component uses as a string that will be read in.

BareDOM offers default styles but also offers tokens that can be used to theme the web component. And it has a special hidden attribute that can be used to hide a web component if needed. These are all declared here.

The next step is to build the shadow DOM once. What is a shadow DOM you ask? It basically is a separate DOM tree that you can attach to an element on a web page. It encapsulates the css styles within the component.

Note that if the code refers to `du/...` then that is a shared namespace where all DOM utilities are defined.


```html
(defn- init-dom! [^js el]
   (let [root  (.attachShadow el #js {:mode "open"})
         style (.createElement js/document "style")
         box   (.createElement js/document "span")
         slot  (.createElement js/document "slot")]
     (set! (.-textContent style) style-text)
     (du/set-attr! box attr-part part-box)
     (.appendChild box slot)
     (.appendChild root style)
     (.appendChild root box)
     (du/mark-initialized! el k-initialized?)))

 (defn- ensure-shadow! [^js el]
   (when-not (du/initialized? el k-initialized?)
     (init-dom! el)))
 ```

 Note the use of `^js` here, If you do not type hint every parameter that is a DOM node, then Closure Advanced build will rename the property and then the code will break.

 The next step is the render pipeline. This is the core of the web component. It follows a three-phase process, READ, CHECK, APPLY. Read-model reads in the attributes and ensures they are normalised.

 ```html
 (defn- read-model [^js el]
    (model/normalize
     {:size-raw       (du/get-attr el model/attr-size)
      :color-raw      (du/get-attr el model/attr-color)
      :label-raw      (du/get-attr el model/attr-label)
      :label-present? (du/has-attr? el model/attr-label)}))
  ```

  Apply-model! uses named helpers to change an aspect of the DOM if needed. The changes are made and when done the model is cached.

  ```html
  (defn- apply-model! [^js el m]
   (ensure-shadow!  el)      ;; Helper to build the shadow DOM the first time
   (apply-css-vars! el m)    ;; Writes the size and color of the icon as css custom properties on the host element
   (apply-aria!     el m)    ;; Makes the icon meaningful for assistive tech like a screen reader
   (du/setv! el k-model m))
   ```

   Once done, the next step is to CHECK if the new model is different from the cached model. It ensures nothing is updated until it is actually needed.

   ```html
   (defn- update-from-attrs! [^js el]
   (let [new-m (read-model el)
         old-m (du/getv el k-model)]
     (when (not= old-m new-m)
       (apply-model! el new-m))))
    ```

  By using the `du` utilities we work with a single way to manipulate attributes etc, and an additional benefit is that we can link a trace recorder there so that updates can later be followed and debugged if needed. We are almost ready now to finish the web component, the last thing to dos is write the lifecycle functions that each web component needs. Since this is shared across all web components in the library, we can write these functions once and reuse them everywhere:

  ```html
  (defn- connected! [^js el]
    (update-from-attrs! el))

  (defn- attribute-changed! [^js el _name old-val new-val]
    (when (not= old-val new-val)
      (update-from-attrs! el)))

  (defn- install-property-accessors! [^js proto]
    (du/install-properties! proto model/property-api))

  (defn init! []
    (component/register! model/tag-name
                         {:observed-attributes  model/observed-attributes
                          :connected-fn         connected!
                          :attribute-changed-fn attribute-changed!
                          :setup-prototype-fn   install-property-accessors!}))
   ```

   The component is registered using its name, the observed attributes, and 3 functions that create and update the component. `Connected-fn` is called the first time the component is rendered to set its attributes, and `attribute-changed-fn` is called every time something needs updating, e.g. an attribute gets a value, is changed or goes away. `Setup-prototype-fn` adds the javascript properties `size`, `color, and `label`.

   The `components/register! function does the heavy lifting. It creates the custom element that represents the web component and gives that to the browser.

   ```html
   (defn register!
   [tag-name class-opts]
   (when-not (.get js/customElements tag-name)
     (.define js/customElements tag-name (make-element-class class-opts))))
   ```
   The function is simple, because the heavy lifting is done in the `make-element-class` where the options map is turned into a class.

   ```html
   (defn make-element-class
     "Create a custom element class from a declarative options map.

      Required keys:
        :observed-attributes  — #js [...] array of attribute names
        :connected-fn         — (fn [el] ...) called on connectedCallback
        :attribute-changed-fn — (fn [el name old new] ...) called on attributeChangedCallback

      Optional keys:
        :disconnected-fn      — (fn [el] ...) called on disconnectedCallback
        :moved-fn             — (fn [el] ...) called on connectedMoveCallback. A move with
                                moveBefore calls neither connected-fn nor disconnected-fn.
        :form-associated?     true to mark as form-associated element. Its formDisabledCallback
                              arrives as a change of its disabled attribute, with a lifecycle
                              record of that change. The old and new
                              values of that change are not the attribute's: a fieldset writes
                              no attribute.
        :form-reset-fn        — (fn [el] ...) called on formResetCallback
        :setup-prototype-fn   — (fn [proto] ...) install properties/methods on prototype
        :internal?            — when true, skip firing the dev-tool lifecycle hook
                                on every callback. Set this on elements that are
                                themselves part of the dev tooling (e.g. the
                                x-trace-history dock); otherwise their own
                                connect/disconnect/attribute records would
                                pollute the trace they produce."
     [{:keys [observed-attributes
              connected-fn
              disconnected-fn
              moved-fn
              attribute-changed-fn
              form-associated?
              form-reset-fn
              setup-prototype-fn
              internal?]}]
     (let [klass (js* "(class extends HTMLElement {})")
           ;; ^js keeps connectedMoveCallback, which the externs do not name, through the
           ;; release build.
           ^js proto (.-prototype klass)
           fire! (if internal? (fn [_]) fire-lifecycle-hook!)]

       (set! (.-observedAttributes klass) observed-attributes)

       (when form-associated?
         (set! (.-formAssociated klass) true))

       (set! (.-connectedCallback proto)
             (fn []
               (this-as ^js this
                 (fire! {:type :lifecycle/connected :el this})
                 (connected-fn this))))

       (when disconnected-fn
         (set! (.-disconnectedCallback proto)
               (fn []
                 (this-as ^js this
                   (fire! {:type :lifecycle/disconnected :el this})
                   (disconnected-fn this)))))

       (set! (.-connectedMoveCallback proto)
             (fn []
               (this-as ^js this
                 (fire! {:type :lifecycle/moved :el this})
                 (when moved-fn
                   (moved-fn this)))))

       (set! (.-attributeChangedCallback proto)
             (fn [n o v]
               (this-as ^js this
                 (fire! (change-record this [n o v]))
                 (change! attribute-changed-fn this [n o v]))))

       (aset proto hold-key
             (fn [f]
               (this-as ^js this
                 (with-held-changes! attribute-changed-fn this f))))

       (when form-associated?
         (set! (.-formDisabledCallback proto)
               (fn [d]
                 (this-as ^js this
                   (fire! (change-record this (disabled-change d)))
                   (change! attribute-changed-fn this (disabled-change d))))))

       (when form-reset-fn
         (set! (.-formResetCallback proto)
               (fn [] (this-as ^js this (form-reset-fn this)))))

       (when setup-prototype-fn
         (setup-prototype-fn proto))

       klass))
   ```

The class form is written with the following declaration `(js* "(class extends HTMLElement {})")`. Then it adds several static fields to the class and installs the lifecycle callbacks defined earlier. There are a few other elements that are interesting.

For example, we use a `hold-key` to prevent attribute changes from happening until the hold is released. As a result, many attribute changes can be made before the component is rendered. If a single change throws an error, it is reported, and the next attribute is changed.

When everything is added to the class, we call `setup-prototype-fn.` This method is called once with the prototype and here the new web component adds its properties and methods. When done, the code returns the class.

## Web components are not that simple

As you can see, a lot of work must be done to be able to generate icons in a web page. As the components get more complex, the code will obviously need to handle that complexity (e.g. a form, a button that can be pressed etc.). The process however is pretty much the same every time so you can re-use a lot of boiler plate code that is in the shared code base for BareDOM.

## What do you get for it?

The web component is truly small (the model and source file contribute about 2kb to a bundle) and can be reused regardless of framework. It will work in vanilla js, angular, react, solid, svelte and vue.

## What isn't covered here

In another post I will try to explain some of the challenges you might run into when using web components and how they can be addressed. Things like server-side state, efficient re-rendering and debugging.

## Source code
If you want to see more code, then everything is available in the [BareDOM](https://github.com/avanelsas/baredom) web component library. it currently holds more than 100 web components, and you can see every one of them in the demo that is on the accompanying GitHub page [here](https://avanelsas.github.io/baredom/).
