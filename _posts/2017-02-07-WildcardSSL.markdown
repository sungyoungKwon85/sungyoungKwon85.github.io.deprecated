---
layout: post
title:  "Wildcard SSL certification"
date:   2017-02-07 13:10:53 +0900
categories: SSL
---

참조 : https://www.securesign.kr/guides/kb/30
<br><br>


와일드카드(Wildcard) 인증서란, 기본 도메인의 하위 서브 도메인에 SSL 적용을 무제한 적용할수 있는 인증서 입니다. 최근에 SSL 인증서 발급 약 30% 선이 Wildcard SSL 인증서로 발급이 되고 있으며, 이는 그만큼 효용성이 높다는 것을 증명합니다.

Wildcard SSL 도 SAN 인증서의 일종이며, RFC 국제 표준 X.509 확장 기술 입니다.  웹브라우져에서 인증서 상세보기 항목중 [주체 대체 이름 - DNS Name] 항목에서 기본 도메인과 서브 도메인 와일드카드가 포함되어 있음을 확인할 수 있습니다.

참고해 보실 문서 : https://en.wikipedia.org/wiki/Wildcard_certificate  (Wildcard certificate - 위키피디아 영문)

Wildcard SSL 인증서에는 1개 Wild * 도메인만 포함이 가능했으나, 최근에는 여러개의 Wildcard 도메인도 SAN  포함이 가능해졌습니다.  여러개의 도메인에 Wildcard 적용이 필요한 경우 최적의 인증서 입니다.



[주요 활용 용도]

서브 도메인 각각 발급 받는것보다 Wildcard SSL 적용이 비용절감에 유리한 경우
1개 인증서로 여러개의 서브 도메인에 SSL 을 적용하고자 하는 경우
웹서버에서 443 SSL 기본 포트로 모든 서브 도메인 웹사이트에 적용하고자 하는 경우
서브 도메인으로 여러대의 웹서버 운영시 각 웹사이트에 SSL 적용하고자 하는 경우
