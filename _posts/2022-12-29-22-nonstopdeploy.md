---
layout: post
title:  "Spring actuator를 활용한 무중단 배포"
date:   2022-12-29 00:01:05
categories: CICD
tags: spring deploy actuator
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*Spring actuator를 활용한 무중단 배포 프로세스 예시 (with Nginx)*

---

**서비스 교체 프로세스**

1. 서비스하는 살아있는 포트 체크(/actuator/health)
2. A가 살면 B를 시작
3. B완료 후 B를 보는 nginx upstream으로 스위칭
4. 일정 슬립 후(기존응답 대응) A의 shutdown(/actuator/shutdown) 호출

**이중화 프로세스**

1. A만 보는 nginx upstream으로 스위칭
2. B재기동 처리 및 health응답 대기
3. B만 보는 nginx upstream으로 스위칭
4. A재기동 처리 및 health응답 대기
5. AB보는 nginx upstream으로 스위칭

---

등록 서비스 종류가 적으면 service 등록해서 처리하는 것도 무난할 듯 하나 단순 프로세스 재 기동 처리나 프로세스 생존 여부는 아래 참조해도 될 듯하다.

**재기동 스크립트**

```bash
vi restart.sh

# restart.sh
ps -ef | grep "some-application" | grep -v grep | awk '{print $2}' | xargs kill -9 2 > /dev/null
if [ $? -eq 0 ];then
    echo "some-application Stop Success"
else 
    echo "some-application Not Running"
fi

echo "some-application Restart!"
echo $1
nohup $JAVA_HOME/bin/java -jar /deploy/some-application.jar --spring.profiles.active=dev > /dev/null 2>&1 &
```

**권한 부여**

```bash
chmod 755 restart.sh
```