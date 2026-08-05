{
  "title": "Onix"
}

---

# Onix
A *Multilingual Toolchain & Runtime* that is build on top of the [The WebAssembly Component Model](https://component-model.bytecodealliance.org/introduction.html) and [Cranelift](https://cranelift.dev/).

## Why We Need It
Languages have libraries that are basically rewrites of one another (e.g. [Python datetime](https://docs.python.org/3/library/datetime.html), [JS Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date))
maintaining two libraries that do the same thing seems like wasted effort if we had one library both langauges could use.
Wasm can potentially solve this problem, suppose we compiled a implementation of the datetime library into a Wasm module.
If two languages implemented some standardized interface for the Wasm module we could compile both languages to
Wasm and now both can use the same library in a Wasm runtime.

Maintaing a interface into the shared library Wasm module requires much less effort than maintaing the source code for a library.
Also the source code of a Wasm module can be written in any language which allows slower scripting languages
to call into optimized libraries for performance gains (I know this is naive and will take a lot of work to become a reality).
