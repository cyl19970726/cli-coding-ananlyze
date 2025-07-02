# 1. Agent的基本原理
![[Pasted image 20250702190426.png]]

https://platform.openai.com/docs/guides/function-calling?api-mode=responses

![[Pasted image 20250702190532.png]]
https://github.com/cyl19970726/continue-reasoning/blob/main/packages/ai-research/function-call/agent.ts


## 2. cli-coding实现中要解决的关键问题
1. 如何调用LLM
2. 如何构建tool并行执行系统
3. 如何构建执行历史管理
4. 如何读取文件
	1. 
	2. glob tool
	3. grep tool
	4. read tool 
5. 如何编辑代码
	1. https://github.com/cyl19970726/continue-reasoning/blob/main/packages/agents/contexts/coding/tests/runtime.test.ts
	2. applyEditBlock
	3. ApplyWholeFile
	4. ApplyRangeEdit
	5. applyUnifiedDiff
6. 怎么进行版本管理
7. 怎么设计权限模块
	1. https://github.com/cyl19970726/continue-reasoning/blob/main/packages/agents/contexts/coding/sandbox/seatbelt-sandbox.ts
8. 如何撰写系统提示词
https://github.com/cyl19970726/continue-reasoning/blob/main/packages/agents/coding-agent.ts
	  coreSystemPrompt
     codingGuidelines
        + taskManagerGuidelines
        + responseGuidelines
        + toolUsageGuidelines
        + environmentContext;

