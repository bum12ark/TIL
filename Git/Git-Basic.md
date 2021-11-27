# Setting

---

## 계정 연결

```bash
git config --global user.name "bum12ark"
git config --global user.email "bum12ark@gmail.com"
```

- `--global`: 해당 명령어가 없을 경우 하나의 리파지토리에서만 이용하겠다는 뜻

## allias

- 깃 명령어를 짧게 사용하기 위해서 사용

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all"
```

- allias 및 현재 Git 설정 상태 보기: `git config --list`

## GitHub에 코드 올리기

---

- **GitHub?**
    - 온라인 코드 저장소
    - 무료
    - 많은 오픈소스들이
- Github login → repository 생성

```bash
cd ~
mkdir git-exer
cd git-exer
# README.md 파일 생성
echo "Country와 함께하는 Git 교실" >> README.md
git init
git add README.md
git commit -m "initial commit"
# 원격 저장소와 연결
git remote add origin https://github.com/bum12ark/git-exer.git
git push -u origin master
```

## GitHub에서 코드 받기

---

### clone

- 원격 저장소에 있는 코드를 내려 받는 것
- 실무에서 일하면서 새로운 repository를 만들어서 올리는 것보다 이미 다른 사람이 만든 것을 clone 하는 경우가 더 많다.

### 강사 repository clone 실습

- [https://github.com/HwangNara/git-class](https://github.com/HwangNara/git-class)

```bash
cd ~
git clone https://github.com/HwangNara/git-class
cd git-class
```

### Vue repository (오픈 소스) 실습

- [https://github.com/vuejs/vue](https://github.com/vuejs/vue)

```bash
git clone https://github.com/vuejs/vue
cd vue
echo "Vue에 코드 기여" >> my.md
git add my.md
git commit -m 'Add my.md'
git push
???
```