
Package Scripts
- `npm version <major|minor|patch>` - Package version 하나 올림
	- `preversion` script 돌림
		- Run test, and add any files that should be added to the commit (`git add`)
	- `postversion` script 돌림
		- File system cleanup, automatically push commit, etc (`git push && git push --tags && rm -rf build/temp`).

`npm publish` - NPM으로 배포

TS Compiler Options for Writing a Library
https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html#im-writing-a-library

https://cmdcolin.github.io/posts/2022-05-27-youmaynotneedabundler