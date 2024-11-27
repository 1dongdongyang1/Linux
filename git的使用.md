# git的使用

[toc]

---

## 1.什么是git

**概念**：

+ Git 是一种**分布式版本控制系统**，广泛用于管理和追踪代码文件的更改。
+ 它由 Linus Torvalds 于 2005 年创建，最初是为了更好地管理 Linux 内核的开发。
+ Git 提供了强大的功能来跟踪代码变化、协作开发、合并代码以及在多个分支上并行开发，是现代软件开发中的重要工具之一。

## 2.主要特点

1. **分布式**：每个开发者的本地存储库都包含了完整的代码历史和版本信息，这意味着可以在没有网络连接的情况下进行提交、分支操作等
2. **高效**：Git 使用快照（Snapshot）机制来存储变化，而不是逐个文件地记录差异，因此速度非常快，且节省存储空间。
3. **支持分支和合并**：Git 中的分支是轻量级的，可以轻松创建和删除，非常适合并行开发。合并工具也非常强大，可以处理复杂的代码合并场景。
4. **可靠性**：Git 使用 SHA-1 哈希算法来确保数据的完整性和安全性。所有数据在存储时都会进行哈希处理，从而防止意外或恶意的更改。

## 3.基本概念

1. **工作区（Working Directory）**：存储当前项目的文件，开发者在这里进行代码的修改。
2. **暂存区（Staging Area / Index）**：用于暂存被标记为“已准备提交”的文件和更改。它类似一个缓存区，帮助开发者组织即将提交的内容。
3. **本地存储库（Local Repository）**：用于保存提交的记录，包括所有提交的版本历史。开发者可以在本地存储库中进行提交、回滚等操作。
4. **远程存储库（Remote Repository）**：通常用于团队协作，它存储在服务器上，开发者可以将本地更改推送（push）到远程仓库，或者从远程仓库拉取（pull）更新。

## 4.基本操作

### 4.1**初始化和配置**

+ 初始化 Git 仓库：

  ```bash
  git init
  ```

+ 配置用户信息：

  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "your.email@example.com"
  ```

  上述的用户信息在Linux操作系统下，将被存储在用户家目录的`.gitconfig`文件里

### 4.2**文件的跟踪和提交**

+ 将文件添加到暂存区：

  ```bash
  git add filename
  ```

  或者添加所有更改的文件：

  ```bash
  git add .
  ```

+ 提交更改到本地存储库：

  ```bash
  git commit -m "Commit message"
  ```

  需要注意的是，提交信息需要写清楚，因为提交信息可以被人预览，而且删除不了

### 4.3**分支操作**

+ 创建新分支：

  ```bash
  git branch branch_name
  ```

+ 切换到指定分支：

  ```bash
  git checkout branch_name
  ```

+ 创建并切换到新分支：

  ```bash
  git checkout -b new_branch_name
  ```

### 4.4查看状态和历史

+ 查看当前状态：

  ```bash
  git status
  ```

+ 查看提交历史：

  ```bash 
  git log
  ```

### 4.5与远程仓库交互

+ 将本地仓库关联到远程仓库：

  ```bash
  git remote add origin https://github.com/yourusername/yourrepository.git
  ```

+ 推送本地分支到远程仓库：

  ```bash
  git push origin branch_name
  ```

+ 从远程仓库拉取最新的更改：

  ```bash
  git pull origin branch_name
  ```

## 5.工作流程示例<p id="5.0"></p>

一个常见的 Git 工作流程如下：

1. **克隆仓库**（如果已有远程仓库）：

   ```bash
   git clone https://github.com/yourusername/yourrepository.git
   ```

2. **创建和切换分支**：在本地创建新分支，并切换到新分支进行开发。

   ```bash
   git checkout -b feature_branch
   ```

3. **修改并提交**：进行代码修改，将更改添加到暂存区并提交。

   ```bash
   git add .
   git commit -m "Add new feature"
   ```

4. **推送到远程分支**：将本地分支推送到远程仓库以便进行协作。

   ```bash
   git push origin feature_branch
   ```

5. **合并分支**：在完成开发后，将分支合并到主分支。

   ```bash
   git checkout main
   git merge feature_branch
   ```

   