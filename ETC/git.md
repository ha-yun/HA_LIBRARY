### fork해온 레포 원본 메인 pull해오는 방법
1. upstream 리모트 추가 확인하기
- 먼저, 원본 레포지토리(upstream)가 리모트로 추가되어 있는지 확인
    `git remote -v`

2. upstream 리모트 추가하기
- upstream을 아직 추가하지 않았다면, 아래 명령어로 원본 레포지토리(upstream)를 추가
    `git remote add upstream https://github.com/원본레포지토리/레포지토리명.git`

3. upstream에서 최신 변경 사항 Pull 하기
- upstream에서 최신 변경 사항을 로컬로 가져오려면, fetch 명령어를 사용하여 upstream의 최신 브랜치를 가져옴.
    `git fetch upstream`

4. 로컬 브랜치에 병합하기
- fetch 후, upstream의 브랜치(주로 main 또는 master)의 최신 내용을 로컬 브랜치에 병합
    ```
        git checkout main  # 또는 작업 중인 브랜치
        git merge upstream/main  # 또는 upstream/master
    ```
- 이 명령은 upstream의 main 브랜치에 있는 최신 변경 사항을 로컬 main 브랜치에 병합

5. 변경사항 Push하기 (선택사항)
- 로컬에서 upstream의 변경사항을 성공적으로 병합한 후, 포크한 레포지토리(origin)로 푸시할 수 있음
    `git push origin main  # 또는 작업 중인 브랜치`

6. pull request


