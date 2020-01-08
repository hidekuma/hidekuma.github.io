---
title: "Vim: sync clipoard to WSL"
excerpt_separator: <!--more-->
categories:
  - Vim
  - WSL
tags: 
  - wsl
  - synchronize
  - clipboard
  - vim
---

# Vim on WSL: synchronize system clipboard
WSL 자체에서 <kbd>shift</kbd> + <kbd>c</kbd> 와 <kbd>shift</kbd> + <kbd>v</kbd>로 복사/붙여넣기 클립보드 기능을 지원하나, 빈 문자열 까지 같이 복사된다.

또한, `tmux`로 나뉘어진 `pane`까지 침범하여 복사가 되곤 하여 너무 불편하였다.
`vim`의 `yank`기능을 클립보드에 전달하기 위해, 아래의 코드를 `.vimrc`에 넣어주면 쉽게 해결 가능하다.
<!--more-->

```bash
let s:clip = '/mnt/c/Windows/System32/clip.exe' 
if executable(s:clip)
    augroup WSLYank
        autocmd!
        autocmd TextYankPost * call system('echo '.shellescape(join(v:event.regcontents, "\<CR>")).' | '.s:clip)
    augroup END
end
```
