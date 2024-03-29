---
layout: post
title:  "까먹는 명령어 메모"
date:   2022-11-01 22:00:00
categories: ETC
desc: "가끔 필요한데 까먹어서 찾게되는 명령어들 모아보기"
tags: memo
cover: note.png
---

#### Linux

**1. 파일 중 특정 문자를 포함한 라인을 전부 출력한다.**

```bash
sudo grep -hnr "Direct Request" application.log > ~/output.txt  
```

**2. 날짜 지난 파일 삭제(feat. crontab)**

```bash
0 0 * * * find {FolderPath} -ctime +7 -exec rm -f {} \;
```

**3. 확장자 일괄 수정**

```bash
rename 's/.png/.jpg/' *
```

**4. CURL 파일첨부(까먹을 때가 많음)**

```bash
curl -XPOST -H 'content-type:application/octet-stream' 'localhost/uploads?fileName=test.txt&fileExt=.txt' --data-binary @test.txt
```

**5. pid 확인**

```bash
netstat -nao | findstr {port}
jps -v
ps -ef | grep {xxx}
lsof -i :{port}
```

**6. task 확인**

```bash
tasklist | findstr {pid}
```

**7. heap dump**

```bash
jmap -dump:format=b,file=heap.hprof pid
java Xms2g -Xmx4g -XX:+PrintGCDetails -verbose:gc -Xloggc:./log/gc.log -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./log -jar {jar파일명} # jar 실행시
```

**8. 웹 서버 관련**

-  SELinux 폴더권한 403 ([참조][link])
```bash
chcon -Rt httpd_sys_conmtent_t {폴더}
```

- Nginx proxy 허용
```bash
setsebool -P httpd_can_network_connect true
```



---

#### Vim

**1. 포매팅 정렬**

```bash
gg=G
```

**2. 문자 치환**

```bash
:%s/foo/bar         # default
:%s/foo/bar/i       # case 구분 X
:%s/foo/bar/g       # row의 전체
:%s/foo/bar/c       # 묻기모드
:%s/foo/bar/gic     # 전부적용
```

---

#### Git

**1. 유저 정보 초기화 & 저장**

ldap이 수시로 바뀌어서 귀찮을 때가 있어서 메모
```bash
git config --system --unset credential.helper # 초기화
git config credential.helper store # 저장
```

---

#### PowerShell

**1. 포트 점유 PID 찾기 및 kill 처리**

```bash
netstat -ano | findstr :6100

#ex) 8736 PID Kill
taskkill /F /PID '8736'
````

[link]: https://www.lesstif.com/system-admin/selinux-httpd_sys_rw_content_t-boolean-24445086.html
