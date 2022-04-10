## How to build robust Vue 3 components

Vue has been my go-to framework for the last couple of years. It gets the job done, it‘s easy to use, and I understand it quite well. Over time, I had built up some great boilerplate code to bootstrap new projects. The scaffold project had `TypeScript` support, used `vue-class-component`, `vuex`, `vue-router` and had also set up some neat tooling stuff. Every new app started off with this template, which I was very happy with. The only problem: That was Vue 2.x.

Vue 3 has officially been released on 18 September 2020, and [as of 7 February 2022, it‘s now the default version](https://twitter.com/vuejs/status/1490592213184573441?s=21&t=Dpb5-9HxF-PIPqb1dxmBoQ). So when I started working on a new app in autumn 2021, an app which I knew would exist for many years, I thought it would be a good idea to start with Vue 3 right away. But since Vue 3 brings quite some changes — most notably, the Composition API — I had been very hesitant to make the switch. Especially, because my beloved scaffold project had worked so well.

But kicking off a large project with an old framework doesn‘t make any sense, so I needed to figure out a way to get started with Vue 3. I struggled a bit to get into the „flow“ of building clean, maintainable and robust components first, but I think I found a way that I probably like even more than the Vue 2.x class component way. So in this post, I want to share my opinionated approach to work with Vue 3. An approach, that will hopefully help you building better Vue components. We will create a component step-by-step, explaining all the decisions we make in detail. But first, let‘s get started with some fundamental things.

## 1. TypeScript

Starting with Vue 3, TypeScript is now a first-class citizen of the framework (Vue 3 is written in TypeScript itself). For that reason, it's especially easy to use TypeScript in your Vue 3 projects. I had been using TypeScript in Vue 2 already, and starting with Vue 3, it makes even more sense. [Use Vue with TypeScript](https://vuejs.org/guide/typescript/overview.html#using-vue-with-typescript), your entire codebase becomes type-safe and it will save you from a lot of troubles.

## 2. Composition API

As mentioned before, the [Composition API](https://vuejs.org/guide/extras/composition-api-faq.html) is probably the most important concept to understand in comparison to Vue 2. In Vue 2, we only had the Options API (where we define `data`, `props`, `methods`, and so on in our components), where in Vue 3, we *also* have the more flexible Composition API, that exposes reactive data and logic, which can be reused in other components as well.

I‘d argue that it generally makes sense to use the Composition API for new Vue 3 projects, as it‘s a lot more versatile and allows for easier sharing of logic and data. In fact, I haven‘t written an Options API-style component in Vue 3 yet, but we‘re getting there. ✌️

## 3. Syntax

Now that we‘ve committed ourselves to prefer the Composition API over the Options API, we need to choose one of the syntax styles that are supported to create Composition API components.

**❌ Option 1 — Component** (example from the official Vue.js documentation)

The first option is to export a default component and expose reactive data using the `setup` method. When I first saw this, I thought that it was very verbose. Luckily, there‘s a better way — but let's see the conventional syntax first.

```javascript
<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)

    return {
      count
    }
  }

  mounted() {
    console.log(this.count) // 0
  }
}
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

**✅ Option 2 — `script setup`**

The second option is to use the [`script setup`](https://vuejs.org/api/sfc-script-setup.html#script-setup) syntax, which has a couple of benefits (less verbose, exposure of top level bindings, better TypeScript support, … I will explain them in detail as we proceed). So this is what we‘re going to use.

```javascript
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

If you compare the `script setup` syntax with the component syntax, you can instantly see a huge difference. It's a lot more clean and concise, and is generally preferred over the "default" component approach, because of its better TypeScript support, performance, and ease of use.

## 4. Putting it together

So if we combine the use of TypeScript and the Composition API (using the `script setup` syntax), we get the following boilerplate code for our Vue 3 components. It's important to use the `lang="ts"` attribute so that strict type checking is enabled.

```javascript
<script setup lang="ts">
</script>

<template>
</template>
```

Quite little code, isn't it? I really value the beauty and simplicity of writing components like this. Compared to the class component approach in Vue 2, there is practically no overhead required to bootstrap new components.

## 5. Custom button component

Let's build our first custom `MagicButton` component in Vue 3. We will add more functionality to the component step-by-step, which I will explain in detail.

Our initial draft renders a default HTML button element with the text "Click me", but doesn't provide any options to pass props or receive events.

```javascript
// MagicButton.vue

<script setup lang="ts">
</script>

<template>
  <button>Click me</button>
</template>
```

### Props

We want to accept a text that the button should display. Therefore, we first add a `Props` interface to the component, which defines the shape of the props that can be passed. We then call `defineProps<Props>()` that exposes our props.

```diff
// MagicButton.vue

<script setup lang="ts">
+ interface Props {
+ text: string
+ }

+ defineProps<Props>()
</script>

<template>
+  <button>{{ text }}</button>
</template>
```

Like that, we are able to import the component and use it as follows.

```javascript
// Home.vue

<script setup lang="ts">
import MagicButton from './components/MagicButton.vue'
</script>

<template>
  <MagicButton text="Login" />
</template>
```

### Default values

We want to add another prop and set a default value if not explicitly set from outside. First, we add another prop called `loading`, which accepts a boolean value. Since we don't want to explicitly set this prop, we use the `withDefaults` compiler macro to define a default value. That way, the compiler knows that this property is optional and won't warn us when not setting it. We also set the `button` state to `disabled` if `loading` is true and show a different text in that case.

```diff
// MagicButton.vue

<script setup lang="ts">
interface Props {
  text: string
+ loading: boolean
}

+ withDefaults(defineProps<Props>(), {
+  loading: false
+ })
</script>

<template>
+  <button :disabled="loading">
+    <span v-if="loading">Loading...</span>
+    <span v-else>{{ text }}</span>
   </button>
</template>
```

### Emits

We also want to get an event when a button is actually clicked. We therefore define emits using the `defineEmits` compiler macro. Whenever the button is clicked, we call the `click` event, passing the actual click event payload.

```diff
// MagicButton.vue

<script setup lang="ts">
interface Props {
  text: string
  loading: boolean
}

withDefaults(defineProps<Props>(), {
  loading: false
})

+ const emit = defineEmits<{
+  (event: 'click', mouseEvent: MouseEvent): void
+ }>()
</script>

<template>
+  <button @click="(event) => emit('click', event)" :disabled="loading">
     <span v-if="loading">Loading...</span>
     <span v-else>{{ text }}</span>
   </button>
</template>
```

### Wrapping up

So we created a custom `<MagicButton>` component and added support to accept props (including default values) using `defineProps` and `withDefaults`, and emit events using `defineEmits` — all in a type-safe manner! That's pretty neat. We can now use our custom component as follows.

```javascript
// Home.vue

<script setup lang="ts">
import MagicButton from './components/MagicButton.vue'

const click = (event: MouseEvent) => {
  console.log('click', event)
}
</script>

<template>
  <MagicButton loading text="Login" @click="click" />
</template>
```

## 6. Custom input component

Let's build a second custom component called `MagicInput`. In this component, we will use a custom `v-model` expression, call functions within our component, as well as introduce reactivity using `ref` and `computed`. Again, we start with the most basic setup, rendering only a default HTML input element.

```javascript
// MagicInput.vue

<script setup lang="ts">
</script>

<template>
  <input type="text" />
</template>
```

### Two-way binding

To use two-way binding with `v-model` in a custom component, we can use the `modelValue` prop and the `update:modelValue` emit accordingly.

```diff
// MagicInput.vue

<script setup lang="ts">
+ interface Props {
+   modelValue: string
+ }

+ defineProps<Props>()

+ const emit = defineEmits<{
+  (event: 'update:modelValue', value: string): void
+ }>()
}
</script>

<template>
+ <input :value="modelValue" @input="emit('update:modelValue', $event.target.value)" type="text" />
</template>
```

We can then use the `v-model` expression on our custom component to two-way bind the text.

```javascript
// Home.vue

<script setup lang="ts">
import { ref } from 'vue'
import MagicInput from './components/MagicInput.vue'

const text = ref('')
</script>

<template>
  <MagicInput v-model="text" />
</template>
```

### ref and computed

To get a bit more into detail on what the Composition API offers us, we are now going to add some reactivity using `ref` and `computed`. We therefore get rid of the two-way binding and instead, accept only an initial value and use an internal `ref` that handles input changes.

```diff
// MagicInput.vue

<script setup lang="ts">
+ import { ref } from 'vue'

interface Props {
- modelValue: string
+ initialValue: string
}

const props = defineProps<Props>()

// Introduce a ref and pass the initalValue prop as the initial value
+ const text = ref(props.initialValue)

- const emit = defineEmits<{
-  (event: 'update:modelValue', value: string): void
- }>()
}
</script>

<template>
  <input v-model="text" type="text" />
</template>
```

So far so good, we now have a reactive `ref` variable that is bound to the `input` using the `v-model` expression. That way, the variable always reflects the current value of the input. Now, let's add a `computed` property that displays the length of the current text.

```diff
// MagicInput.vue

<script setup lang="ts">
- import { ref } from 'vue'
+ import { ref, computed } from 'vue'

interface Props {
  initialValue: string
}

const props = defineProps<Props>()

const text = ref(props.initialValue)
+ const textLength = computed(() => text.value.length)
</script>

<template>
  <input v-model="text" type="text" />
+ <p>Your text is {{ textLength }} characters long.</p>
</template>
```

So now we can use the `textLength` computed property that always returns the current length of the text. Please note that, because `text` is a `ref`, we need to call `.value` (e.g. `text.value.length`) before to get the actual value of the property.

### Functions

Most of the time, we also want to execute some kind of logic within our components, that's why we'll add a function that logs the current text of the input to the console whenever it changes. Since all top-level bindings are exposed to our template, it's as easy as adding a default TypeScript function.

```diff
// MagicInput.vue

<script setup lang="ts">
import { ref, computed } from 'vue'

interface Props {
  initialValue: string
}

const props = defineProps<Props>()

const text = ref(props.initialValue)
const textLength = computed(() => text.value.length)

+ const logText = (text: string) => {
+  console.log(`The current text is ${text}`)
+ }
</script>

<template>
+ <input 
+   v-model="text"
+   @input="logText($event.target.value)"
+   type="text"
+ />
  <p>Your text is {{ textLength }} characters long.</p>
</template>
```

### Wrapping up (again)

This time, we created a custom `<MagicInput>` component that allowed us to use `v-model`, leverage reactivity using `ref` and `computed` and also execute custom logic. We can now use our custom component as follows.

```javascript
// Home.vue

<script setup lang="ts">
import { ref } from 'vue'
import MagicInput from './components/MagicInput.vue'

const text = ref('Ronald Blüthl')
</script>

<template>
  <!-- First version using v-model -->
  <MagicInput v-model="text"></MagicInput>

  <!-- Second version using initial value -->
  <MagicInput initial-value="Ronald Blüthl"></MagicInput>
</template>
```

## 7. Summary

We built two minimalistic components, `MagicButton` and `MagicInput` that cover 95% of the functionality that is typically needed when creating custom components.

- Using TypeScript to enjoy full type safety for all our components 
- `script setup` to make use of the Composition API in a very convenient way
- `defineProps` and `withDefaults` for props and default values
- `defineEmits` for emits (events)
- `v-model` for two-way binding
- `ref` for reactive data
- `computed` for reactive read-only data

Using these few techniques, we're able to pass data to a component, receive data from a component, bind values in both ways and create reactive state and logic. 

## 8. General considerations

In Vue, there are sometimes many ways to achieve the desired result. I usually try to be as consistent as possible, that's why I make sure to follow these rules (most of which are suggested in the Vue documentation as well). 

### Naming and casing

- Use `PascalCase` for components (`<MagicButton>`, `<MagicInput>`)
- Use `kebab-case` for props (`initial-value`, `loading`) — except `modelValue`

### Best practices

- Always extract a `Props` interface instead of defining an inline object
- Prefer `initial-value="Ronald Blüthl"` over `:initial-value="'Ronald Blüthl'"` when NOT passing a variable
- Prefer `loading` over `:loading="true"` when passing boolean props
- Avoid `watch` expressions as much as possible (they are really hard to debug)

### One-way data flow

Data flows into one way only in Vue. If the parent data updates, the child data is updated as well, but not the other way around. This means, that `props` are read-only. If you want to mutate passed props, use a ref and pass the prop value as the initial value, for example:

```javascript
interface Props {
  initialText: string
}

const props = defineProps<Props>()

const text = ref(props.initialText)
```

### Don't mutate objects and arrays

Because of the nature of JavaScript objects and arrays, it is technically possible to mutate them in a component, which then also affects the parent component. You should always avoid this, as it becomes extremely hard to discover problems in your data flow later on. Instead, emit an event with the updated data and let the parent component perform the mutation, for example:

```javascript
const props = defineProps<{
  names: string[]
}>

const emit = defineEmits<{
  (event: 'change', names: string[]): void
}>()

const addName = (name: string) => {
  const updatedNames = [...props.names, name]
  emit('change', updatedNames)
}
```

## 9. Final words

So this is my practical approach to build Vue 3 components, which I used for many projects in the last couple of months. As mentioned at the beginning of the article, I've been using Vue for many years already, and I think that it really grew as a framework. The combination of using the Composition API with TypeScript and `script setup` is a really nice way to write clean and concise components.

I hope there are some tips and best practices you can pick up. If you have any questions, just let me know in the comments below. If you found this post useful, feel free to [follow me on Twitter](https://twitter.com/rbluethl), as I document my journey as an indie hacker there. My DMs are open (I'm always happy to help). 🙌