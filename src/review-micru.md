# micru

`micru` means Master in C/C++ and Rust, a tool that uses AI to translate C/C++ into Rust.

## 功能
1. 增加 `clap`，用于解析命令行参数。看上去更像一个命令行工具，且方便后期扩展命令；
1. 增加 `jwt` 模块，用于统一处理 JWT；
1. 后端抽象，设计 `invoke`, `outcome` 处理平台请求和结果；
1. 结果不再是单一的转换结果代码，还带有 AI 输出的提示信息；
1. 作品思路：不追求翻译的准确性，定位是转译助手，可以提供多个转译结果和提示，辅助用户工作。

