{
  "title": "Onix"
}

---

# Onix
Onix is a *Multilingual Toolchain & Runtime* that is build on top of the [The WebAssembly Component Model](https://component-model.bytecodealliance.org/introduction.html) and [Cranelift](https://cranelift.dev/).

## Why We Need It
Programming languages have package/libraries that are basically rewrites of one another (e.g. [Python datetime](https://docs.python.org/3/library/datetime.html), [JS Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date))
maintaining two libraries that do the same thing seems like wasted effort if we had one library both langauges could use.
Wasm can potentially solve this problem; suppose we compiled a implementation of the datetime package into a Wasm module
if two programming languages `a` and `b` implemented the standardized interface for the Wasm module we could compile both languages to
Wasm and both could use the same implementation in a Wasm runtime.

Maintaing a interface into the Wasm module requires much less effort than maintaing the source code for a library.
Also the source code of a Wasm module can be written in any programming language which allows slower scripting languages
to call into optimized libraries for performance gains.
