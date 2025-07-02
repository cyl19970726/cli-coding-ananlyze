# 系统提示词管理对比分析

## OpenAI Codex CLI vs Gemini CLI vs Aider 提示词架构对比

### 核心架构差异

| 方面 | OpenAI Codex CLI | Gemini CLI | Aider |
|------|------------------|------------|-------|
| **架构模式** | 分离式模块化 | 集成式动态生成 | 分层策略模式 |
| **主要文件** | `codex-rs/core/prompt.md` (静态) | `packages/core/src/core/prompts.ts` (动态) | `aider/coders/*_prompts.py` (类层次) |
| **项目文档** | `AGENTS.md` | `GEMINI.md` | 无特定格式 |
| **覆盖机制** | 环境变量 + 文件层次 | 环境变量完全覆盖 | 配置文件 + 命令行参数 |
| **工具指令** | 独立指令文件 | 集成在主提示词中 | 分 Coder 类型专门化 |
| **编辑格式** | V4A 差异格式 | 全文件替换 | 5种策略 (whole/edit-block/diff/architect/ask) |

### 1. 提示词生成策略

#### OpenAI Codex CLI: 分离式模块化
```
核心提示词 (prompt.md)
├── 工具特定指令 (apply_patch_tool_instructions.md)
├── 项目上下文 (AGENTS.md 层次发现)
├── 环境信息 (动态注入)
└── 用户配置 (~/.codex/instructions.md)
```

**特点**:
- 静态 Markdown 文件作为基础
- 独立的工具指令文档
- 分层配置覆盖机制
- 重点在代码编辑和补丁应用

#### Gemini CLI: 集成式动态生成
```typescript
getCoreSystemPrompt(config: Config): string {
  return `
    # 核心指令
    ${generateCoreInstructions()}
    
    # 工具使用指南  
    ${generateToolInstructions(config.tools)}
    
    # 环境上下文
    ${buildDynamicContext(config)}
    
    # 用户记忆
    ${formatUserMemory(userMemory)}
  `;
}
```

**特点**:
- 单一函数动态构建完整提示词
- 工具指令内嵌生成
- 层级化记忆系统
- 个人化用户偏好持久化

#### Aider: 分层策略模式
```python
# aider/coders/editblock_prompts.py - 编辑器特定提示词
class EditBlockPrompts(CoderPrompts):
    main_system = """Act as an expert software developer.
Always use best practices when coding.
Respect and use existing conventions, libraries, etc that are already present in the code base.
Take requests for changes to the supplied code.

Once you understand the request you MUST:
1. Decide if you need to propose *SEARCH/REPLACE* edits to any files
2. Think step-by-step and explain the needed changes
3. Describe each change with a *SEARCH/REPLACE block*
"""
    
    system_reminder = """# *SEARCH/REPLACE block* Rules:
Every *SEARCH/REPLACE block* must use this format:
1. The *FULL* file path alone on a line
2. The opening fence and code language
3. <<<<<<< SEARCH (exact match required)
4. ======= (divider)
5. >>>>>>> REPLACE (replacement content)
6. The closing fence
"""
```

**特点**:
- 基于继承的分层提示词架构
- 编辑器类型特定的专门指令
- 示例驱动的提示词设计
- 严格的格式验证规则

### 2. 项目文档系统

#### OpenAI Codex CLI: Git 感知的层次发现
```rust
// 搜索优先级
1. 环境变量覆盖
2. 项目本地 CODEX.md  
3. 项目根目录 AGENTS.md (Git 根目录)
4. 用户全局 ~/.codex/instructions.md
5. 系统默认提示词
```

**AGENTS.md 示例**:
```markdown
# Codex Development Guidelines

## Rust Codebase Notes
- 主要是 Rust 代码库，TypeScript 前端
- 注意沙箱限制
- 使用 CODEX_SANDBOX_NETWORK_DISABLED=1

## Development Workflow
1. 修改 codex-rs/ 中的 Rust 代码
2. 使用 cargo build --release 构建
3. 测试 codex-cli/ 中的 TypeScript 集成
```

#### Gemini CLI: 多维度记忆发现
```typescript
// 记忆层次结构
1. 用户全局记忆 (~/.gemini/GEMINI.md)
2. 项目根目录记忆 (project-root/GEMINI.md)
3. 中间目录记忆 (parent-dirs/GEMINI.md)  
4. 当前目录记忆 (cwd/GEMINI.md)
5. 子目录记忆 (subdirs/GEMINI.md)
```

**GEMINI.md 示例**:
```markdown
# Gemini CLI Development Context

## Project Structure
- TypeScript monorepo，使用 npm workspaces
- Core 逻辑在 packages/core/，CLI 在 packages/cli/

## Testing Guidelines
- 使用 Vitest 框架编写测试
- 使用 vi.mock() 模拟外部依赖
- 测试文件与源文件同位置
```

#### Aider: 无特定项目文档格式

```python
# Aider 不依赖特定的项目文档文件
# 而是通过 RepoMap 系统自动分析项目结构

class RepoMap:
    def scan_repository(self, root_path):
        """自动发现项目结构和上下文"""
        # 1. 检测项目类型 (package.json, Cargo.toml, setup.py 等)
        # 2. 使用 Tree-sitter 解析代码结构  
        # 3. 提取符号和依赖关系
        # 4. 生成智能的仓库映射
```

**项目分析示例**:
```
aider/repo_analysis.md (自动生成)
├── Language: Python 
├── Framework: None detected
├── Key Files:
│   ├── setup.py (package configuration)
│   ├── aider/main.py (entry point)
│   └── aider/coders/ (core functionality)
├── Dependencies: litellm, tree-sitter, gitpython
└── Test Framework: pytest
```

**特点**:
- 无需手动维护项目文档
- 自动代码结构分析
- 动态上下文生成
- 基于实际代码的智能推理

### 3. 工具指令管理

#### OpenAI Codex CLI: 独立指令文档
```rust
// 工具定义与指令绑定
FunctionDefinition {
    name: "apply_patch".to_string(),
    description: "Apply changes using V4A diff format",
    instruction_file: Some("apply_patch_tool_instructions.md"),
}
```

**apply_patch_tool_instructions.md**:
```markdown
# Apply Patch Tool Instructions

## V4A Diff Format Specification
@@ file_path @@
--- line_number,count
+++ line_number,count
 context_line
-removed_line
+added_line
```

#### Gemini CLI: 动态工具指令生成
```typescript
function generateToolInstructions(tools: Tool[]): string {
  let instructions = "";
  
  for (const tool of tools) {
    instructions += `
### ${tool.displayName} (${tool.name})
**Purpose**: ${tool.description}
**Safety Level**: ${tool.confirmationRequired ? 'Requires confirmation' : 'Auto-approved'}
`;
  }
  
  return instructions;
}
```

#### Aider: 编辑器类型特定的指令系统

```python
# aider/coders/ - 不同编辑策略的专门指令
class EditBlockPrompts(CoderPrompts):
    """SEARCH/REPLACE 块编辑策略的专门指令"""
    
    example_messages = [
        {
            "role": "user", 
            "content": "Change get_factorial() to use math.factorial"
        },
        {
            "role": "assistant",
            "content": """mathweb/flask/app.py
```python
<<<<<<< SEARCH
from flask import Flask
=======
import math
from flask import Flask
>>>>>>> REPLACE
```"""
        }
    ]

class UnifiedDiffPrompts(CoderPrompts):
    """统一差异格式的专门指令"""
    
    main_system = """Apply changes using unified diff format.
Each diff block should specify:
- File path
- Line numbers  
- Exact changes with +/- markers
"""

class WholeFilePrompts(CoderPrompts):
    """全文件替换的专门指令"""
    
    main_system = """Replace entire file contents.
Always show the complete updated file.
"""
```

**特点**:
- 每种编辑策略有专门的提示词类
- 丰富的示例驱动设计
- 格式验证和错误处理指令
- 基于编辑复杂度的策略选择指导

### 4. 用户个性化

#### OpenAI Codex CLI: 文件配置系统
```markdown
# ~/.codex/instructions.md (全局配置)
- 偏好 TypeScript 而非 JavaScript
- 新函数总是包含单元测试
- 使用描述性变量名
- 遵循清洁代码原则

# CODEX.md (项目特定)
- 这是使用 hooks 的 React 项目
- 使用 styled-components 进行样式设计
- API 调用应使用自定义 useApi hook
```

#### Gemini CLI: 智能记忆系统
```typescript
// 个人记忆持久化
export class MemoryTool {
  async execute(params: MemoryToolParams): Promise<MemoryToolResult> {
    // 解析现有记忆结构
    const memoryStructure = this.parseMemoryStructure(existingMemory);
    
    // 添加新信息到适当类别
    this.addInformationToCategory(memoryStructure, category, information);
    
    // 重新生成记忆文件
    const updatedMemory = this.renderMemoryStructure(memoryStructure);
  }
}
```

**生成的个人记忆**:
```markdown
# Personal Memory for Claude Code

## User Preferences
- 偏好函数式编程风格
- 总是要求类型安全
- 喜欢详细的错误处理

## Code Patterns & Preferences  
- 使用 async/await 而非 Promise.then()
- 偏好明确的接口定义
- 组件应该有 PropTypes 或 TypeScript 类型
```

#### Aider: 配置文件驱动的个性化

```yaml
# .aider.conf.yml - 项目级配置
model: gpt-4o
weak-model: gpt-4o-mini
edit-format: diff

# 行为偏好
auto-commits: true
dirty-commits: false
attribute-author: true

# 代码风格偏好
language: english
dark-mode: true
pretty: true

# 模型特定配置
max-tokens: 8000
temperature: 0.1
```

```python
# aider/args.py - 多层配置系统
class ConfigManager:
    def resolve_config(self):
        """配置优先级: 命令行 > 环境变量 > 项目配置 > 用户配置 > 默认值"""
        
        config = {}
        
        # 1. 默认配置
        config.update(self.default_config)
        
        # 2. 用户全局配置 (~/.aider.conf.yml)
        user_config = self.load_user_config()
        config.update(user_config)
        
        # 3. 项目配置 (.aider.conf.yml)
        project_config = self.load_project_config()
        config.update(project_config)
        
        # 4. 环境变量 (AIDER_*)
        env_config = self.load_env_config()
        config.update(env_config)
        
        # 5. 命令行参数 (最高优先级)
        cli_config = self.parse_cli_args()
        config.update(cli_config)
        
        return config
```

**特点**:
- 多层次配置文件系统
- YAML 格式的声明式配置
- 环境变量支持 (AIDER_* 前缀)
- 项目和用户级别的分离配置

### 5. 环境感知能力

#### OpenAI Codex CLI: 系统级上下文注入
```rust
pub async fn build_context_prompt(&self) -> Result<String> {
    // 1. 基础系统信息
    // 2. Git 仓库信息  
    // 3. 沙箱环境检测
    // 4. 项目类型检测
}
```

#### Gemini CLI: 动态环境分析
```typescript
export async function buildDynamicContext(config: Config): Promise<string> {
  // 1. 系统环境信息 (平台、Node 版本、内存)
  // 2. Git 仓库状态 (分支、状态、远程 URL)
  // 3. 项目分析 (类型、语言、框架、构建工具)
  // 4. 沙箱环境 (激活状态、限制、网络访问)
}
```

#### Aider: 智能仓库分析和上下文生成

```python
# aider/repomap.py - 智能上下文分析
class RepoMap:
    def generate_context(self, files_to_edit):
        """为编辑任务生成智能上下文"""
        
        context = []
        
        # 1. 项目元信息
        project_info = self.analyze_project_structure()
        context.append(f"Project Type: {project_info.type}")
        context.append(f"Language: {project_info.language}")
        context.append(f"Framework: {project_info.framework}")
        
        # 2. 相关文件分析
        related_files = self.find_related_files(files_to_edit)
        for file in related_files:
            symbols = self.extract_key_symbols(file)
            context.append(f"\n## {file}")
            context.extend([f"- {symbol}" for symbol in symbols])
        
        # 3. 依赖关系
        dependencies = self.analyze_dependencies(files_to_edit)
        if dependencies:
            context.append("\n## Dependencies")
            context.extend(dependencies)
            
        # 4. 最近变更历史
        recent_changes = self.get_recent_git_changes()
        if recent_changes:
            context.append("\n## Recent Changes")
            context.extend(recent_changes)
            
        return "\n".join(context)
        
    def extract_key_symbols(self, file_path):
        """使用 Tree-sitter 提取关键符号"""
        symbols = []
        
        # 解析代码结构
        tree = self.parse_file(file_path)
        
        # 提取函数、类、导入等
        for node in tree.root_node.children:
            if node.type in ['function_definition', 'class_definition']:
                name = self.get_symbol_name(node)
                importance = self.calculate_importance(node)
                symbols.append((name, importance))
                
        # 按重要性排序
        symbols.sort(key=lambda x: x[1], reverse=True)
        return [name for name, _ in symbols[:10]]  # 取前10个最重要的
```

**特点**:
- 基于 Tree-sitter 的深度代码分析
- 智能符号提取和重要性排序
- 动态依赖关系分析
- Git 历史集成和变更感知

### 6. 提示词模板系统

#### OpenAI Codex CLI: 用户指引模板
```markdown
# Prompting Guide

## Request Complexity Levels

**Small Requests**: 
codex "fix the TypeScript error in utils.ts"

**Medium Tasks**:
codex "create a user registration form with validation"

**Large Projects**:
codex "build a complete todo app with React, Node.js backend"
```

#### Gemini CLI: 智能工具推荐
```typescript
export function generateContextualToolRecommendations(
  userQuery: string,
  projectContext: ProjectContext
): string {
  // 基于查询内容推荐工具
  if (userQuery.includes('file')) {
    recommendations.push('Consider using read_file or read_many_files tools');
  }
  
  // 基于项目类型推荐
  if (projectContext.framework === 'React') {
    recommendations.push('For React projects, consider component-level changes');
  }
}
```

#### Aider: 基于策略的智能工具选择

```python
# aider/main.py - 智能编辑策略选择
def choose_edit_format(main_model, edit_format_arg):
    """基于模型能力和任务复杂度选择编辑策略"""
    
    if edit_format_arg:
        # 用户明确指定
        return edit_format_arg
        
    # 基于模型能力自动选择
    if main_model.supports_unified_diff():
        return "diff"  # 最精确，适合复杂编辑
    elif main_model.supports_edit_blocks():
        return "edit-block"  # 平衡精确性和易用性
    else:
        return "whole"  # 兜底策略，适合简单场景

def generate_strategy_guidance(chosen_strategy, project_context):
    """为选定策略生成使用指导"""
    
    guidance = []
    
    if chosen_strategy == "diff":
        guidance.append("Use unified diff format for precise changes")
        guidance.append("Include sufficient context lines for uniqueness")
        
    elif chosen_strategy == "edit-block":
        guidance.append("Use SEARCH/REPLACE blocks for targeted edits")
        guidance.append("Ensure SEARCH sections exactly match existing code")
        
    elif chosen_strategy == "whole":
        guidance.append("Replace entire file contents")
        guidance.append("Preserve all existing functionality unless explicitly changed")
        
    # 添加项目特定指导
    if project_context.language == "python":
        guidance.append("Maintain consistent indentation (spaces/tabs)")
        guidance.append("Preserve import order and grouping")
        
    return "\n".join(guidance)
```

**特点**:
- 基于模型能力的动态策略选择
- 项目上下文感知的指导生成
- 策略特定的最佳实践提示
- 自适应的复杂度匹配

## 总结对比

### OpenAI Codex CLI 的优势
1. **专业化**: 专注于代码编辑和补丁应用的专业工具链
2. **模块化**: 清晰的关注点分离，便于维护和扩展
3. **稳定性**: 基于静态文件的配置，减少动态错误
4. **精确性**: V4A 差异格式提供精确的代码编辑能力

### Gemini CLI 的优势
1. **智能化**: 动态环境感知和智能工具推荐
2. **个性化**: 深度的用户偏好学习和记忆系统
3. **灵活性**: 完全动态的提示词生成和上下文适应
4. **全面性**: 层级化记忆发现和综合项目分析

### 设计哲学差异

**OpenAI Codex CLI**: "专业工具，精确执行"
- 专注于代码修改和执行任务
- 强调安全性和精确性
- 适合专业开发者的日常编码工作

**Gemini CLI**: "智能助手，全面服务"  
- 提供全方位的软件工程支持
- 强调学习和适应用户习惯
- 适合各种复杂度的开发任务

### Aider 的优势
1. **多策略灵活性**: 5种不同编辑策略适应各种场景
2. **智能化程度**: Tree-sitter 驱动的深度代码理解
3. **开源生态**: 完全开源，社区驱动发展
4. **Git 原生集成**: 深度的版本控制工作流集成

**Aider**: "多策略智能，开放生态"
- 基于编辑复杂度的智能策略选择
- 深度代码理解和仓库级别的智能分析
- 开源驱动的快速迭代和社区贡献

### 提示词管理进化趋势

1. **从静态到动态**: Codex 的静态文件 → Gemini 的动态生成 → Aider 的智能适应
2. **从通用到专门化**: 单一提示词 → 工具特定指令 → 策略特定的专门提示词类
3. **从手动到自动**: 手工编写 → 模板生成 → 基于代码分析的智能生成
4. **从简单到复杂**: 基础指令 → 示例驱动 → 多维度上下文感知

三个系统都展现了现代 AI 辅助开发工具在提示词管理方面的先进实践，代表了不同的发展路径和设计理念，为 CLI 编程助手的发展提供了宝贵的参考。