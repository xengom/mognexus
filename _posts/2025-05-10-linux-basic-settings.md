---
layout: post
title: 리눅스 초기설정 정리 
date:   2025-05-10
last_modified_at: 2025-05-10
category: [Etc]
tags: [Linux, Shell, zsh]
---
<br/>
# 리눅스 초기 설정 
개인적으로 여러 리눅스 배포판을 설치하면서 필수적으로 셋팅하는 것들을 정리하기 위해 작성하였다.
*Arch Linux + xfce4를 기준으로 작성되었음. 우분투 등 데비안 계열은 pacman/yay 대신 apt/apt-get 사용할 것*

## 1. 언어 설정
배포판에 따라 다르겠지만 언어를 한국어로 설정하고 설치하는 경우 '사진', '다운로드' 등 기본 폴더명이 한국어로 생성된다. ~~그리고 영어가 더 간지난다.~~

### locale 설정
```bash
sudo vi /etc/locale.gen
```
상기 파일에서 `en_US.UTF-8`과 `ko_KR.UTF-8`의 주석을 제거 후 저장한다.

## 입력기 설정(ibus)
### ibus 설치
```bash
sudo pacman -S ibus ibus-hangul
```
### profile 설정
```bash
sudo vi ~/.xprofile
```
```bash
# .xprofile
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus

ibus-daemon -drx
```

실행 명령어인 `ibus-daemon -drx`를 입력하면 ibus를 통해 입력할 수 있다

# 2. Shell 설정
## zsh 설치
```bash
sudo pacman -S zsh 
sudo chsh -l
sudo chsh -s /usr/zsh
```
`chsh -l`로 설치되어있는 쉘 목록을 확인하고 `chsh -s`로 사용할 쉘의 경로를 지정해준다.

## oh-my-zsh 설치
```bash
curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh
```
## plugin 설치 및 설정
편의성을 위해 자동완성과 하이라이트 관련 플러그인을 추가한다.
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

```bash
# ~/.zshrc
plugins=(
    git
    zsh-autosuggestions
    zsh-syntax-highlighting
)
```

## neovim 설치 및 설정
```bash
sudo pacman -S neovim
```
```bash
# ~/.zshrc
export EDITOR=nvim
export VISUAL=nvum
alias vi='nvim'
alias vim='nvim'
```

## 3. Albert 설치(Optional)
macOS의 spotlight 역할을 해주는 프로그램이다.
```bash
yay -S albert
```
```bash
cp /usr/share/applications/albert.desktop ~/.config/autostart/
```

## 4. nvm 설치(Optional)
JS 개발 시 필요한 node.js 버전을 관리하는 매니저이다.
```bash
yay -S nvm
```
### 에러 발생 시 조치
```bash
Your shell must be initialized befor nvm will function correctly.
Run the following, and consider adding it to ~/.bashrc or ~/.zshrc:
    source /usr/share/nvm/init-nvm.sh
```
상기 에러 발생 시 다음과 같이 조치한다.
```bash
echo 'source /usr/share/nvm/init-nvm.sh' >> ~/.zshrc
source ~/.zshrc
```

### node 설치
특정 버전이 필요하지 않다면 다음 명령어로 최신 lts 버전의 node를 설치한다.
```bash
nvm install --lts
```

