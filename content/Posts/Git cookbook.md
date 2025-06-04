---
title: Git cookbook
draft: 
tags:
  - dev
  - ps1
  - post
socialDescription: A walkthrough about what is git and the basic command with simple examples for local repository management and integration with remote repository like github
socialImage: https://rodcordeiro.github.io/shares/img/gitflow.jpg
---
# TL;DR;
> A walkthrough about what is git and the basic command with simple examples for local repository management and integration with remote 

Hey Devs, Today I'm gonna show you my notes about Git. I'm studing it and made this post to have my notes somewhere I can take a look and share with some friends, so for a deeper look about it, visit the [documentation](https://git-scm.com/docs/)

So, first of all...

# What is Git?

Git is a file source-control management(SCM) system created by Linus Torvalds at 2005 and the core maintainer nowadays is Junio Hamano. Through it we're able to develop projects where many people can contribute simultaneously, editing and creating files, allowing they to exists without the risk of it's changes to be overwritten. **Important:** _Note that **git** and **github** are different things!_

# How it works?

Basically:

- You have the repository, that is the main folder;
- The files has 3 stages (_not staged, staged and commited_);
- You can separate the repository on branchs;

So, imagine you have a project where you’re going to use git. We’re gonna call it as `Example` (pretty creative huh?!) and the project is under the folder example. So, you have the folder and all the files of the project and then initialize the repository (see below how to do it). Now, all files are on _not staged_ state, or as git shows: `not tracked changes`, so everything in our folder is not tracked by git. Let’s track them [so](https://www.notion.so/Git-the-cookbook-396388b948554c85b1d6eb39fe8e4744?pvs=21)! After files are succesfully added to the file index, git does a simple task everytime a change is made: Check if the file is already tracked, where in the file the change is made, and store it. So everytime we change something, write or delete, git tell us that we modified the file and ask to save this to file index, what allow us to use the branches to separate our work in multiple ‘lines’ without conflicting any of them, because every time we merge the work of them, the git does this job and when it find some file that were modified on both sides, it shows us where is the conflict and merge the rest of files, combining the changes. **Note** that as _conflict_, I mean, when a change were made on same line of the same file on both branches or sides.

# Installing and configuring git

## Installing

To install git on Windows access [http://git-scm.com/](http://git-scm.com/), on Linux (_debian-based)_ execute `sudo apt install git` on terminal, for CentOS use `sudo dnf install git`

## Configuring git

After installing it, there's a lot of configurations available to do, but to start you just need to tell it who you are. For this, you need to setup a name and email, what can be done running these commands:

`git config --global user.name 'YOUR NAME'`

`git config --global user.email 'your@email.com'`

Now, let's start.

# Handling files with git

## Starting a repository

You can initialize or clone a repository. To start an repository you must execute `git init`. If the repository already exists, you can clone it with `git clone REPOSITORY_URL`.

_The syntax for every git command will be `git COMMAND -flags`_

## Tracking changes

Everytime you create/edit/delete a file you must add this change to the file index of the repository, what means, stage this change. This is a pretty simple task, just execute `git add .`. **Note:** This command accepts a lot of values, you can pass just one file to add (`git add index.md` for example), you can git an file to exclude. On this example we use the `.` that means to add all the changes, the same as `git add all`.

After the changes has been properly added to the index, we must commit them, an analogy would be to say that adding them we're preparing the changes to be saved and commiting is saving it.

## How to commit?

The syntax is pretty simple (`git commit`). For basic commits, you can just use the flag `-m` that allows you to pass a message on the command directly. So, for example, on our first commit, we could just use the `git commit -m "First commit"`. To give a better commit message, just use the command without flags, it will open a predefined text editor (you can change it with the `git config --global core.editor "PATH TO EDITOR EXECUTABLE"`).

## Branchs

The git works with branches, that works like layers and can exists simultaneously even if totally different of each other.

> **Example:** You join an Open Source project, to develop a software. You go to the source repository of the software, clone it so you can work on it and be part of the project. After cloning you will have your repository clone with a master branch, that is, with a single branch of work. In order to work and keep up to date with the source repository, that is, work on top of an updated version, you must create a new branch of work, where you will develop your part without overwriting (unless purposely) someone else job.

**NOTE**: The default branch is the _master_ or _main_. This can be changed and sometimes may be defined by the company git flow.

The basic commands for git branch is:

`git branch BRANCH_NAME` to create a new branch

`git branch -d BRANCH_NAME` to delete the branch

`git branch` to list the existing branchs

To switch between the branchs, you use the `git checkout` command. It can be used to create a new branch too.

`git checkout -b NEW_BRANCH` to create a new branch and switch to it

`git checkout BRANCH` to swith between branchs.

When working with branchs, it's common to maintain on the master branch the released version, and creating a new branch to develop a new feature. (read about _git flow_)

![Atlassian / Merging vs Rebasing](https://wac-cdn.atlassian.com/dam/jcr:01b0b04e-64f3-4659-af21-c4d86bc7cb0b/01.svg?cdnVersion=1606)

Atlassian / Merging vs Rebasing

So, imagine you've a project, you're developing a new feature on the _feature_ branch. After you've finished, you must merge it on the master, that contain the completed and tested version.

For this you have 2 ways, the **merge** and **rebase**.

## Merge and Rebase

Both merge and rebase has a similar workflow to integrate the changes on another branch (this case: the master branch), the difference between them is how they handle the commit history where one keeps the history and the others move the entire history.

So, you’ve been working on the project “mobileApp”, and you created a new branch for a feature. Once you finished this feature, you have commited all its files and changes, you will have 2 branches (_master_ and _feature_) with a history of commits and changes each other, and maybe different files if you created some on feature branch. When you merge it, it’s like: _You copy all files from feature branch and overwrite it on master branch, this create a merge commit with all commits indexes._ The difference when rebasing is that it overwrites all files, takes the history and _put in front of the actual history_ and the merging just _link the history point where it were merged,_ keeping the

### Referencias

[https://www.atlassian.com/git/tutorials/merging-vs-rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

[https://dev.to/vitordangelo/git-e-github-controle-e-compartilhe-seu-codigo-1l4d](https://dev.to/vitordangelo/git-e-github-controle-e-compartilhe-seu-codigo-1l4d)

[https://dev.to/maxpou/git-cheat-sheet-advanced-3a17](https://dev.to/maxpou/git-cheat-sheet-advanced-3a17)

[https://blog.rocketseat.com.br/iniciando-com-git-github/](https://blog.rocketseat.com.br/iniciando-com-git-github/)

[https://github.com/rafaballerini/GitTutorial](https://github.com/rafaballerini/GitTutorial)

[https://opensource.com/article/20/11/git-aliases](https://opensource.com/article/20/11/git-aliases)

[https://opensource.com/article/20/10/advanced-git-tips](https://opensource.com/article/20/10/advanced-git-tips)

[https://opensource.com/article/20/7/git-best-practices](https://opensource.com/article/20/7/git-best-practices)

[https://linuxize.com/post/change-git-commit-message/](https://linuxize.com/post/change-git-commit-message/)

**Merge**

**Rebase**

**Git Pull**

**Git Push**

Git Stash

# Git flow

---

tags:

- git
- code title: gitflow revision: 15/09/22 revised by: Rodrigo Mendonça

---

> Git flow é uma sugestão de modelo para controle de branches, visando facilitar a organização e mensuramento da evolução do projeto.

---

## Estrutura de branches

### Branches padrões

Estas branches serão padrão em todos os projetos e por tanto, devem ser criadas no momento que o repositório é inicializado. Através desta estrutura, as políticas para garantir a organização do código serão aplicadas.

- **main:** branch principal, a partir desta branch será feita a build para produção
- **develop:** branch base para desenvolvimento, deve estar sempre atualizada ou a frente da main. A partir desta as outras branchs serão criadas.

### Padronização de nomenclatura das branches

A padronização de nomenclatura das branchs auxília na organização do desenvolvimento de novas funcionalidades e correções, além de facilitar a aplicação de métricas e o mensuramento da evolução do projeto. A padronização segue sempre `TIPO/descricao-breve-ou-numero-do-card`. Os tipos possíveis são:

- **fix:** branch para correções. _Única branch que pode ser criada a partir da main quando o erro for identificado em produção._
- **feat:** branch para desenvolvimento de novas funcionalidades.
- **docs:** branch para desenvolvimento ou atualização da documentação do projeto.
- **chore:** branch para atividades que não sejam relacionadas diretamente a correções ou desenvolvimento de novas funcionalidades, como ajustes de configurações, atualizações de dependências, etc., algo que não altere diretamente o código, como ferramentas auxíliares ou processo de build.
- **ci:** branch para implementação ou atualização de configurações e/ou funcionalidades referentes a CI/CD.
- **test:** branch para desenvolvimento, implementação ou atualização de testes.
- **refactor:** branch para a refatoração do código.

---

## Padronização de commits

A padronização da estrutura de branchs auxilia na organização do repositório e do histórico, mas não é tudo para garantir um histórico limpo e organizado, a padronização das mensagens de commit ajudam a cumprir esta meta e facilitar o dia-a-dia. O por quê padronizar? Imagine que você está atuando em um projeto, ele foi para homologação porém por intempéries da vida você precisou atuar em outros projetos e só voltou a mexer nele após alguns dias, semanas, meses. Tendo um padrão nos commits, fica bem mais simples de "ler" o histórico e identificar o que havia sido feito e o que ainda precisa ser feito.

### Sintaxe padrão de commit

O padrão a ser seguido deverá ser sempre: `Prefixo: Mensagem sucinta do que foi feito.Closes #numero_do_card` Através deste padrão, teremos o seguinte:

- A indicação sobre o que este commit está implementando ou alterando;
- A mensagem descrevendo o que foi feito e está incluso neste commit;
- A vinculação do card ao commit através do `Closes #numero_do_card`. Este _Closes_ é [uma funcionalidade especial do Azure DevOps](https://docs.microsoft.com/pt-br/azure/devops/repos/git/resolution-mentions?view=azure-devops) e pode ser substituido por `Fix`ou `Fixes`.

### Prefixos permitidos

- **fix:** Utilizado para correções.
- **feat:** Implementação de novas funcionalidades.
- **docs:** criação ou atualização de documentações.
- **chore:** utilizado para atividades que não sejam relacionadas diretamente a correções ou desenvolvimento de novas funcionalidades, como ajustes de configurações, atualizações de dependências, etc., algo que não altere diretamente o código, como ferramentas auxíliares ou processo de build.
- **ci:** Implementação ou atualização de configurações e/ou funcionalidades referentes a CI/CD.
- **test:** Desenvolvimento, implementação ou atualização de testes.
- **refactor:** Refatoração do código.

### Exemplos

> Feat: Adicionado modal na tela xyz. Closes #18177

> Fix: Corrigida a função de pesquisa da tela xpto. Fixes #18177

> Chore: Atualizado o arquivo .env. Closes #18177

> CI: Atualizado arquivo para pipeline de build. Fix #18177

### git & github

![20220616_003057.jpg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8913c430-f706-49bc-b0b1-636d8f2ac361/20220616_003057.jpg)

What is github

Connect to github

validate connection

[https://community.codenewbie.org/yuridevat/git-commands-im-using-in-my-current-project-which-make-me-feel-like-a-pro--115l](https://community.codenewbie.org/yuridevat/git-commands-im-using-in-my-current-project-which-make-me-feel-like-a-pro--115l)

[https://ohshitgit.com/](https://ohshitgit.com/)