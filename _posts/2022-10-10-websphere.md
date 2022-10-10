---
title: "[Websphere] 웹스피어Websphere 8.5에서 관리자 콘솔Application Console 포트 번호 알아내기"
excerpt: ""

categories:
  - Websphere
tags:
  - Websphere
  - 금융IT
  - WAS
 
last_modified_at: 2022-10-10T13:00:00
toc: true
toc_sticky: true
typora-root-url: ../
---



# Websphere Administrative Console port number

운영서버 관리자 콘솔 접근 url은 인수인계를 받았었는데 개발서버 관리자 콘솔은 아무도 알지 못하더라. 만약 그 포트에 방화벽이 뚫려있지 않으면 방화벽 보안정책 신청을 해야해서 Websphere 설정을 뒤져보았다. 결론은 아래와 같다

```
# 기본값
{WAS_ROOT}/profiles/Dmgr0*/Adminagent0*/config/cells/*Cell0*/virtualhosts.xml
HostAlias_4 => 비보안 관리포트
HostAlias_5 => 보안 관리포트

# 기본값으로 접근이 안된다면 이 경로에서 확인 및 수정해서 사용
{WAS_ROOT}/profiles/Dmgr0*/Adminagent0*/config/cells/*Cell0*/nodes/*Node0*/serverindex.xml

WC_adminhost => 비보안 관리포트
WC_adminhost_secure => 보안 관리포트
```

위 두 경로를 뒤져보면 포트 번호가 명시되어있다. 난 첫 번째 경로에 나와있는 포트로 접근이 안되어 두 번째 경로에서 확인하였다. **WC_adminhost**로 검색되는 것이 많을 수가 있는데 그중 아무 포트로 접근하면 되는 것 같다. 난 WC_adminhost로 검색했을 때 최상단에 있는 태그에서 방화벽 뚫여있는 포트로 수정했더니 관리 콘솔로 접근이 되었다.
