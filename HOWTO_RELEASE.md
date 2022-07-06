
# git flow를 사용한 브랜치와 릴리즈 생성

**git-flow** 도구를 사용하여 프로젝트의 브랜치들을 생성, 관리하고 릴리즈를 생성합니다.

**참조문헌**:

- http://danielkummer.github.io/git-flow-cheatsheet/
- https://gitversion.readthedocs.io/en/latest/git-branching-strategies/gitflow-examples/
- http://nvie.com/posts/a-successful-git-branching-model/

## Git flow 설정

```git flow```가 아직 설치되지 않은 경우는 다음과 같이 로컬 환경에 설치하고 설정합니다:

- Git flow를 설정하기 전에 프로젝트는 먼저 ```main``` 브랜치와 ```develop``` 브랜치가 생성되어 있어야 합니다.
- 로컬 개발 프로젝트 폴더에서 최초에 한번 ```git flow init``` 명령을 수행하여 Git flow 설정을 하십시오. 대부분 기본 설정을 그대로 따르되 (즉, 기본 값 제안에 엔터키를 치면 됨), 메인 브랜치 이름은 `master`가 아니라 `main`으로
설정되도록 하십시오.
- 그리고 프로젝트의 기본 브랜치는 ```develop``` 브랜치로 설정하십시오.

**주의**: 릴리즈 후에는 ```main``` 브랜치의 내용은 최신 태그의 내용과 일치하게 됩니다.
반면에 ```develop``` 브랜치 (그리고 ```support/x.y``` 브랜치들) 는 피처 브랜치들과 버그 픽스 브랜치들의
기반 브랜치가 됩니다.  

## 유지보수 (support) 브랜치

이전 버전들을 유지보수 하는 브랜치들을 갖고자 할 때에는 ```git flow support``` 명령을 사용합니다.
이 유지보수 브랜치의 기반 브랜치로 특정 태그나 커밋 해시를 사용하고자 할 때에는 다음과 같은 명령을 사용합니다.

```
git flow support start x.y tag-x.y.z
```

위의 명령은 ```tag-x.y.z``` 태그로부터, ```support/x.y``` 라는 브랜치를 생성하게 됩니다.

예를 들어, 만일 ```develop``` 브랜치의 프로젝트 버전이 ```2.0.0-SNAPSHOT``` 이라면,
그리고 만일 버전 ```1.x``` 범위의 유지보수 브랜치를 생성하고자 한다면,
```support/1.x``` 브랜치를 버전 1.5.0의 태그로부터 생성하면 됩니다.
  
Git flow는 단지 브랜칭 베스트 프랙티스를 자동화한 도구이기 때문에,
태그나 릴리즈 생성 시 유효한 버전 이름 (즉, `-SNAPSHOT`이 없는 버전 이름) 은 반드시
별도로 변경하여 커밋하고 푸시해야 합니다.
    
## Git flow를 사용한 릴리즈 과정

### 릴리즈 과정 시작

```git flow release start ...``` 명령을 사용하여 임시 릴리즈 브랜치를 생성합니다.

이 명령은 많은 경우 ```develop``` 브랜치를 기반으로 하거나, 유지보수 릴리즈인 경우에는
```support/x.y``` 브랜치를 기반으로 수행합니다.

```
git flow release start x.y.z develop
```

이 명령을 수행하고 나면, 자동으로 새로운 임시 릴리즈 브랜치인 ```release/x.y.z``` 브랜치로
이동하게 됩니다.

### 릴리즈 버전 설정

이제 적절한 릴리즈 버전 이름을 다음 예와 같이 pom.xml 파일들에 설정해야 합니다.

```
mvn versions:set -DgenerateBackupPoms=false -DnewVersion="x.y.z"
```

이제 변경사항을 커밋합니다.

```
git commit -a -m "<ISSUE_ID> releasing x.y.z: set version"
```

### 릴리즈 완료

이제 릴리즈를 완료하려면, 아래와 같은 명령을 사용합니다.

```
git flow release finish x.y.z
```

이 과정에서는 Git flow가 자동으로 릴리즈 태그를 생성하고, 임시 릴리즈 브랜치에서 변경한 내용을 ```main``` 브랜치와 ```develop``` 브랜치에 머지합니다.
그리고 마지막으로 임시 릴리즈 브랜치를 삭제합니다.

즉, 위 예제에서 임시 생성된 ```release/x.y.z``` 로컬 브랜치가 삭제됩니다.
또한 이 명령 수행 후, 자동으로 ```develop``` 브랜치로 이동하게 됩니다.

### develop 브랜치에서 버전 올리기

위의 릴리즈 완료 과정에서 임시 릴리즈 브랜치의 모든 변경 사항은 로컬 환경의
```main``` 브랜치와 ```develop``` 브랜치에 자동으로 머지되었습니다. 따라서 이후에
반드시 ```main``` 브랜치와 ```develop``` 브랜치의 커밋을 원격에 푸시해야만 합니다.

그 전에 ```develop``` 브랜치의 커밋들을 원격에 푸시하기 전에, 프로젝트 버전 이름을
다음 개발 사이클을 위해 아래 예와 같이 버전 번호를 올려야 합니다.

```
mvn versions:set -DgenerateBackupPoms=false -DnewVersion="x.y.z+1-SNAPSHOT"
```

### main 브랜치와 develop 브랜치를 원격에 푸시하기

위에 설명된 대로 모든 변경 사항들은 로컬 환경에서 머지되었습니다.
이제 ```git push origin main```과 ```git push origin develop``` 명령으로 모든 커밋을 푸시하십시오.

### 생성된 릴리즈 태그를 원격에 푸시하기

위에서 설명한 릴리즈 완료 과정에서, 릴리즈 버전을 위한 태그가 로컬 환경에서 하나 생성됩니다. 예: ```x.y.z```

참고로 모든 기존 태그들은 다음 명령으로 조회 가능합니다:

```
git tag -l
```

**참고사항**: 아래 명령은 다른 사람들이 생성한 태그들도 보여주도록 모든 태그들을 다시 조회합니다.

```
git fetch --all --tags --prune
```

이제 다음 명령을 수행하여 로컬 환경에서 생성된 태그를 원격에 푸시합니다.

```
git push origin x.y.z
```

이제 릴리즈 공정이 완료되었습니다.
생성된 릴리즈는 태그로 만들어졌고 원격에 퍼블리시된 것입니다.

### 참조 문헌

- [Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
