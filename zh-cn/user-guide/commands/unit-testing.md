# 单元测试

阅读 http://kent-boogaart.com/blog/using-the-visual-studio-test-runner-for-mobile-development

别模拟 ReactiveCommand。

ReactiveCommand 本身是围绕可测试性设计的。 此外，您将通过 Moq 正确地模拟 ReactiveCommand 语义的可能性相当低，这是一个非常复杂的类（如果你这样做，你最终会做一些不必要的工作）。