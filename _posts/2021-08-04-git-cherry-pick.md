---
title: "Git 명령어 알아보기: git cherry pick"
categories:
  - Develop
tags:
  - git
toc: true
toc_label: "Contents"
toc_sticky: true
---

### git cherry pick 알아보기

![](https://res.cloudinary.com/practicaldev/image/fetch/s--kLArrWHH--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://kolosek.com/content/images/2018/05/git-cherry-pick.png)

* 때때로 특정 브랜치의 커밋 중 일부가 필요한 경우가 있다. ~~가령 오늘처럼 release 되지 않은 수정 사항 중에 일부를 당장 적용해야 하는 경우라거나...~~ 이럴 때 `git rebase`나 `git cherry-pick`을 사용한다. 오늘은 ~~바로 오늘 사용한~~ `git cherry-pick`에 대해 알아보고자 한다.

#### git cherry-pick

* 다른 브랜치에 있는 커밋을 선택적으로 내 브랜치에 적용시킨다. 명령어는 아래와 같다.

```bash
git cherry-pick <commit-hash>
```

* 위의 그림을 예시로 생각해 보자.
  *  `feature` 브랜치에서 `master`의 커밋 `C`를 적용하지 않으면 개발이 불가능한 상황이라고 하자. `D`는 아직 개발 중이므로 적용되어서는 안 된다. 이럴 때 `cherry-pick`을 사용하면 `C` 만을 가져올 수 있다. 
  * 커밋 `C`의 해시가 `1234abc`라고 할 때, 이 커밋을 `feature` 브랜치에 가져와 보자.

```bash
# 당연히 현재 브랜치는 feature 여야 한다.
git cherry-pick 1234abc
```

* 위의 명령어를 적용해 주면 아래처럼 `C`가 `feature`에 추가된다. 매우 간단하다. `git log`를 보면 추가된 커밋이 가장 상단(최신)에 있는 것을 확인할 수 있다.

![](https://res.cloudinary.com/practicaldev/image/fetch/s--5lCKUNxi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://kolosek.com/content/images/2018/05/git-cherry-pick-1.png)



* 명령어 뒤에 해시를 여러 개 나열해 주면 여러 개를 한꺼번에 현재 브랜치로 가져올 수 있다. (충돌이 없는 한) 커밋의 순서는 상관없다.

```bash
git cherry-pick 1234abc 5678def 9012ghi
```

*  `<hash1>..<hash3>`을 이용해 hash1에서 hash3 사이의 모든 커밋을 가져올 수 있다. 시간 상으로 hash1이 가장 처음이고, hash3이 이후의 커밋이다. hash1에는 `cherry-pick`이 적용되지 않는다.

#### --continue --abort

* `cherry-pick` 하려는 커밋과 내 브랜치 사이에 conflict이 발생할 수 있다.

##### conflict 고치고 --continue

* 코드를 수정한다.
* `git add`로 수정된 코드를 올린다. *커밋은 할 필요 없다*!
* `git cherry-pick --continue` 명령어로 다시 진행한다. (dash 2개 사용 주의)

##### --abort

* `git cherry-pick --abort`로 `cherry-pick`을 중단하면 이전 상태로 돌아간다.

#### +) git cherry-pick merge

* `merge` 커밋을 `cherry-pick` 하려면 아래 명령어를 사용한다.

````bash
git cherry-pick -m 1 <commit-hash>
````



#### References

* [https://dev.to/neshaz/cherry-picking-with-git-4pmj](https://dev.to/neshaz/cherry-picking-with-git-4pmj)
* [https://cselabnotes.com/kr/2021/03/31/56/](https://cselabnotes.com/kr/2021/03/31/56/)