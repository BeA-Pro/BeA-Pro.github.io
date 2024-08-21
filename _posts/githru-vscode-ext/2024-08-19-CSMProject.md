---
title: Githru-vscode-ext CSM이 만들어지는 과정
author: BeAPro
date: 2024-08-19 13:00:00 +0900
categories: [Web Projects,Githru-vscode-ext]
tags: [Githru-vscode-ext, OSSCA]
image:
  path: /assets/img/title-image/githru.png
  alt: githru
---
## **CSM Dict이 만들어지는 전반적인 과정**

```js
// 1. 로컬에 설치된 git 실행파일 경로 찾아오기(extension.ts)
const gitPath = (await findGit()).path;
```
```js
// 2. 분석할 워크스페이스 주소 가져오기(extension.ts)
1. const currentWorkspaceUri = vscode.workspace.workspaceFolders?.[0].uri;
```
```js
// 3. gitPath와 워크스페이스 주소를 사용해 로그 목록 가져오기(extension.ts)
const gitLog = await getGitLog(gitPath, currentWorkspacePath);

// 4. gitPath와 워크스페이스 주소를 사용해 config 가져오기(extension.ts)
const gitConfig = await getGitConfig(gitPath, currentWorkspacePath, "origin");

// 5. Git 리포지토리의 원격 URL에서 소유자(owner)와 리포지토리(repo) 이름을 추출(extension.ts)
const { owner, repo: initialRepo } = getRepo(gitConfig);
const repo = initialRepo[0];

// 6. analysis engine 객체 생성 (extension.ts)
const engine = new AnalysisEngine({
  isDebugMode: true,
  gitLog,
  owner,
  repo,
  auth: githubToken,
  baseBranchName,
});

// 7. analyzeGit 메서드 실행
const { isPRSuccess, csmDict } = await engine.analyzeGit();

// 8. git log를 활용해 CommitRaws 생성 (index.ts의 analyzeGit 메서드)
const commitRaws = getCommitRaws(this.gitLog);

// 9. commitRaws를 활용해 commitDict 생성 (index.ts의 analyzeGit 메서드)
const commitDict = buildCommitDict(commitRaws);

// 10. pr 정보 가져오기... 근데 아무것도 안가져옴 (index.ts의 analyzeGit 메서드)
const pullRequests = await this.octokit.getPullRequests().catch((err) => {
  console.error(err);
  isPRSuccess = false;
  return [];
}).then((pullRequests) => {
  console.log("success, pr = ", pullRequests);
  return pullRequests;
});

// 11. commitDict를 활용하여 stemDict 생성 (index.ts의 analyzeGit 메서드)
const stemDict = buildStemDict(commitDict, this.baseBranchName);

// 12. commitDict, stemDict, branchName, pullRequests를 활용하여 csmDict 생성 (index.ts의 analyzeGit 메서드)
 const csmDict = buildCSMDict(commitDict, stemDict, this.baseBranchName, pullRequests);

// 13. (extension.ts)
return mapClusterNodesFrom(csmDict);
```

## **

```
commit 51327898979c6bdc5f22d3ae618118d55258bb9f 419c1b2d4a7e55112956447a3da6b25ac0fdd02e
Author:     YongHyunKing <yhkim8246@gmail.com>
AuthorDate: Fri May 10 19:37:45 2024 +0900
Commit:     YongHyunKing <yhkim8246@gmail.com>
CommitDate: Fri May 10 19:37:45 2024 +0900

    fix : change login, join api

0	1	src/pages/Main.js
2	2	src/services/api.js

```

```
commitMessageType =
'fix'
committer =
{name: 'YongHyunKing', email: 'yhkim8246@gmail.com'}
committerDate =
Fri Aug 16 2024 17:32:13 GMT+0900 (한국 표준시)
differenceStatistic =
{totalInsertionCount: 41, totalDeletionCount: 115, fileDictionary: {…}}
id =
'b2ba01d5550933d6c65f05149ca54b5283bb77af'
message =
'fix: 환자 정보와 환자 이송기록 통합'
parents =
(1) ['224d4b1a4e7bc86982a16b53097c0e7b12ddda95']
sequence =
0
tags =
(0) []
[[Prototype]] =
Object
```

```


```
// commitRaws
author = {name: 'YongHyunKing', email: 'yhkim8246@gmail.com'}
authorDate = Fri Aug 16 2024 17:32:13 GMT+0900 (한국 표준시)
branches = (4) ['HEAD', 'main', 'origin/main', 'origin/HEAD']
commitMessageType = 'fix'
committer = {name: 'YongHyunKing', email: 'yhkim8246@gmail.com'}
committerDate = Fri Aug 16 2024 17:32:13 GMT+0900 (한국 표준시)
differenceStatistic = {totalInsertionCount: 41, totalDeletionCount: 115, fileDictionary: {…}}
id = 'b2ba01d5550933d6c65f05149ca54b5283bb77af'
message = 'fix: 환자 정보와 환자 이송기록 통합'
parents = (1) ['224d4b1a4e7bc86982a16b53097c0e7b12ddda95']
sequence = 0
tags = (0) []
```

```
// commitStem
0 = {base: {…}, source: Array(0)}
base = {commit: {…}, stemId: 'main'}
commit = {sequence: 89, id: '86eb028a888c66afe5d1fd276832eaefab9ffbf7', parents: Array(0), branches: Array(0), tags: Array(0), …}
author = {name: 'YongHyunKing', email: 'yhkim8246@gmail.com'}
authorDate = Wed Jul 17 2024 15:46:33 GMT+0900 (한국 표준시)
branches = (0) []
commitMessageType = ''
committer = {name: 'YongHyunKing', email: 'yhkim8246@gmail.com'}
committerDate = Wed Jul 17 2024 15:46:33 GMT+0900 (한국 표준시)
differenceStatistic ={totalInsertionCount: 24, totalDeletionCount: 0, fileDictionary: {…}}
id ='86eb028a888c66afe5d1fd276832eaefab9ffbf7'
message = 'first commit'
parents = (0) []
sequence = 89
tags = (0) []
[[Prototype]] = Object
stemId ='main'
```