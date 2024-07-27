---
title: Githru-vscode-ext 파헤치기
author: BeAPro
date: 2024-07-25 16:00:00 +0900
categories: [Web Projects,Githru-vscode-ext]
tags: [Githru-vscode-ext, OSSCA]
image:
  path: /assets/img/title-image/githru.png
  alt: githru
---
## **Githru-vscode-ext란?**

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-01.png)

Githru-vscode-ext는 GitHub의 복잡한 commit 히스토리를 **Stem**, **Context Preserving Squash Merge**, **Commit Clustering** 기법을 사용하여 단순화하여 시각적 분석을 도와주는 VSCode 익스텐션이다.

Github 링크 : <https://github.com/githru>

### Stem
Stem은 Git 저장소의 DAG(Directed Acyclic Graph) 표현을 간소화하기 위한 개념이다.
Stem은 특정 커밋의 조상 노드 리스트로, 다중 부모 노드가 있을 때 하나의 부모만 포함한다. 이는 Git의 첫 번째 부모 옵션(first-parent option)과 유사하다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-02.png)

위와 같이 복잡한 구조의 commit 내역을 Stem 기술을 사용하면 4가지 Stem으로 나눌 수 있다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-03.png)

기본적으로 마스터 브랜치에서 시작하여 다른 브랜치의 첫 번째 부모 커밋만 남기고 나머지 부모 노드를 제거한다. 이를 통해 각 브랜치의 단순한 경로만 남게 된다. Stem 기법을 활용해 개발자는 중요한 브랜치와 커밋에 집중할 수 있으며, 전체 개발 히스토리를 개괄적으로 파악할 수 있다.



### Context Preserving Squash Merge (CSM)

CSM은 Git 저장소의 복잡한 DAG 구조를 단순화하면서도 중요한 컨텍스트를 보존하기 위해 제안된 기술이다.
여러 Stem 간의 정보를 축약하여 전체적인 그래프의 복잡성을 줄이고, 각 커밋 간의 중요한 문맥을 보존한다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-04.png)

특정 머지 커밋(CSM-base)과 관련된 모든 부모 커밋(CSM-source)을 하나의 노드로 합치면서 CSM-base에 CSM-source의 정보(작성자, 커밋 메시지, 변경된 파일 목록 등)를 포함시킨다. CSM은 주 브랜치부터 시작하여 가장 최근의 커밋 순으로 적용된다. 이를 통해 중요한 브랜치의 토폴로지를 먼저 보존할 수 있다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-05.png)

CSM을 적용시킨 결과는 위와 같다.

### Commit Clustering

Commit Clustering은 유사한 커밋들을 하나의 클러스터로 묶는 기법으로, 각 클러스터는 여러 커밋을 대표하는 하나의 노드로 표현된다.
CSM이 여러 Stem간의 정보를 축약했다면 Commit Clustering은 단일 Stem 내의 커밋들을 유사성에 따라 분류하여 복잡성을 줄인다.

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-06.png)

커밋의 유사성을 측정하기 위해 `Simple Additive Weighting (SAW)` 모델을 사용했다.

> Githru에는 CSM까지만 적용되었다. Commit Clustering은 [데모 사이트](https://githru.github.io/realm-java-demo/)에서 확인 할 수 있다.
{: .prompt-info}


## **Githru-vscode-ext vscode 코드 분석**

![Desktop](/assets/img/githru-vscode-ext/2024-07-25-07.png)

Githru의 구조는 VSCODE, VIEW, ENGINE으로 나눌 수 있다.

`/pakages/vscode/extension.ts`의 `activate` 함수가 최초의 진입점으로 해당 함수에서 엔진과 뷰를 실행시킨다.

### VSCODE

```ts
// extension.ts
export async function activate(context: vscode.ExtensionContext) {
  const { subscriptions, extensionPath, secrets } = context;
  ...
}
```

ExtensionContext는 VS Code 확장 프로그램 개발 시 사용되는 객체로, 확장 프로그램의 수명 주기와 관련된 정보를 제공하고, 확장 프로그램의 상태를 관리하는 데 도움을 준다.

- subscriptions: Disposable 객체의 배열로, 확장 프로그램이 종료될 때 함께 정리해야 할 리소스들을 관리한다. 이벤트 리스너나 명령어 등록 같은 것을 여기 추가하면 확장 프로그램이 비활성화될 때 자동으로 해제된다.
- extensionPath: 확장 프로그램의 루트 경로를 나타내는 문자열이다. 확장 프로그램의 파일에 접근할 때 유용하다.
- secrets: 민감한 정보를 안전하게 저장하고 액세스할 수 있는 API를 제공한다. 확장 프로그램이 사용자 비밀번호나 API 키와 같은 비밀 정보를 저장해야 할 때 유용하다.

```ts
const disposable = vscode.commands.registerCommand(COMMAND_LAUNCH, async () => {
  // 명령어 실행 시 호출될 비동기 함수
});
```
명령어를 등록하고 실행할 때 사용하는 vscode.commands.registerCommand 메서드를 사용하여 명령어를 정의하고, 이를 context.subscriptions에 추가하여 명령어가 확장 프로그램의 생명 주기에 따라 자동으로 정리되도록 한다.

```ts
// 최초 진입점
export async function activate(context: vscode.ExtensionContext) {
  const { subscriptions, extensionPath, secrets } = context;
  const credentials = new Credentials();
  let currentPanel: vscode.WebviewPanel | undefined = undefined;

  await credentials.initialize(context);

  console.log('Congratulations, your extension "githru" is now active!');
  const disposable = vscode.commands.registerCommand(COMMAND_LAUNCH, async () => {
    try {
      console.debug("current Panel = ", currentPanel, currentPanel?.onDidDispose);
      if (currentPanel) {
        currentPanel.reveal();
        return;
      }
      // gitPath : 로컬에 설치된 git 경로
      const gitPath = (await findGit()).path;

      // 분석할 워크스페이스 경로 가져오기
      const currentWorkspaceUri = vscode.workspace.workspaceFolders?.[0].uri;
      if (!currentWorkspaceUri) {
        throw new WorkspacePathUndefinedError("Cannot find current workspace path");
      }

      const currentWorkspacePath = normalizeFsPath(currentWorkspaceUri.fsPath);

      // GitHub API와 상호작용을 위해 토큰 가져오기
      const githubToken: string | undefined = await getGithubToken(secrets);
      if (!githubToken) {
        throw new GithubTokenUndefinedError("Cannot find your GitHub token. Retrying github authentication...");
      }

      // gitPath에 있는 실행파일을 사용하여 워크스페이스에 있는 git branch 목록 가져오기
      const fetchBranches = async () => await getBranches(gitPath, currentWorkspacePath);

      // 초기 실행시 패널에 띄울 브랜치 정하기
      const fetchCurrentBranch = async () => {

        let branchName;
        try {
            branchName = await getCurrentBranchName(gitPath, currentWorkspacePath)
        } catch (error) {
            console.error(error);
        }

        if (!branchName) {
          const branchList = (await fetchBranches()).branchList;
          branchName = getDefaultBranchName(branchList);
        }
        return branchName;
      };

      const initialBaseBranchName = await fetchCurrentBranch();

      // Engine을 실행하여 InitialBaseBranchName에 있는 git commit 목록 clustering
      const fetchClusterNodes = async (baseBranchName = initialBaseBranchName) => {

        // 현재 작업 중인 리포지토리의 전체 Git 로그를 가져옴. 이 로그에는 모든 커밋의 상세 정보가 포함됨
        const gitLog = await getGitLog(gitPath, currentWorkspacePath);

        // 리포지토리의 Git 설정에서 특정 리모트 URL을 가져옴. 이는 주로 원격 저장소의 URL을 확인하는 데 사용
        const gitConfig = await getGitConfig(gitPath, currentWorkspacePath, "origin");

        // 리모트 URL에서 리포지토리 소유자(owner)와 이름(repo)을 추출
        const { owner, repo } = getRepo(gitConfig);
        // Engine 인스턴스 생성
        const engine = new AnalysisEngine({
          isDebugMode: true,
          gitLog,
          owner,
          repo,
          auth: githubToken,
          baseBranchName,
        });

        // Engine 인스턴스를 활용하여 csmDict 생성
        const { isPRSuccess, csmDict } = await engine.analyzeGit();
        if (isPRSuccess) console.log("crawling PR failed");

        return mapClusterNodesFrom(csmDict);
      };

      // 패널에 들어갈 정보 넣기
      const webLoader = new WebviewLoader(extensionPath, context, {
        fetchClusterNodes,
        fetchBranches,
        fetchCurrentBranch,
      });
      currentPanel = webLoader.getPanel();

      currentPanel?.onDidDispose(
        () => {
          currentPanel = undefined;
        },
        null,
        context.subscriptions
      );

      subscriptions.push(webLoader);
      vscode.window.showInformationMessage("Hello Githru");
    } catch (error) {
      if (error instanceof GithubTokenUndefinedError) {
        vscode.window.showErrorMessage(error.message);
        // Github 로그인이 필요함
        vscode.commands.executeCommand(COMMAND_LOGIN_WITH_GITHUB);
      } else if (error instanceof WorkspacePathUndefinedError) {
        vscode.window.showErrorMessage(error.message);
      } else {
        vscode.window.showErrorMessage((error as Error).message);
      }
    }
  });

  const loginWithGithub = vscode.commands.registerCommand(COMMAND_LOGIN_WITH_GITHUB, async () => {
    const octokit = await credentials.getOctokit();
    const userInfo = await octokit.users.getAuthenticated();
    const auth = await credentials.getAuth();

    await setGithubToken(secrets, auth.token);
    vscode.window.showInformationMessage(`Logged into GitHub as ${userInfo.data.login}`);
    vscode.commands.executeCommand(COMMAND_LAUNCH);
  });

  const resetGithubAuth = vscode.commands.registerCommand(COMMAND_RESET_GITHUB_AUTH, async () => {
    await deleteGithubToken(secrets);
    vscode.window.showInformationMessage(`Github Authentication reset.`);
  });

  // 비활성화 될 때 disposable, loginWithGithub, resetGithubAuth 객체 해제
  subscriptions.concat([disposable, loginWithGithub, resetGithubAuth]);

  myStatusBarItem = vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Left, -10);
  myStatusBarItem.text = "githru";
  // Item 클릭시 COMMAND_LAUNCH 명령어 실행(disposable 실행)
  myStatusBarItem.command = COMMAND_LAUNCH;
  subscriptions.push(myStatusBarItem);

  // update status bar item once at start
  myStatusBarItem.show();
}

// this method is called when your extension is deactivated
export function deactivate() {}
```




