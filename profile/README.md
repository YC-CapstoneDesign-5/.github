# 행정 구조대
> **공공일자리 참여자·출석·급여·4대보험 자료를 모아 교차 검증하고, 제출 서식까지 자동으로 만드는 행정업무 지원 웹 플랫폼**

![행정 구조대 한눈에 보기](https://raw.githubusercontent.com/YC-CapstoneDesign-5/.github/main/docs/images/overview.svg)

## 왜 만드나
지자체 일자리창출과는 매달 약 60개 부서가 제각각 서식으로 보내는 엑셀과 종이 출석부를 손으로 취합하고, 인건비를 계산하고, 수천 명이 섞인 4대보험 고지서에서 참여자분을 골라냅니다. 이 반복 작업을 **검증 가능한 자동화**로 바꾸는 것이 목표입니다.

## 핵심 기능
| # | 기능 | 한 줄 요약 |
| :---: | :--- | :--- |
| 01 | 참여자 통합 관리 | 부서 이동·퇴사 후에도 근무·정산 이력 추적 |
| 02 | 엑셀 자동 취합 | 60개 부서의 다른 서식을 표준 데이터로, 원본 행 위치 보존 |
| 03 | 인건비 계산 | 휴가·조퇴·중도 입퇴사까지 1원 단위 정확 계산 |
| 04 | 4대보험 귀속 대조 | 주민번호 단독이 아닌 다차원 매칭, 퇴사자 환급 추적 |
| 05 | 오류 검증 | 누락·중복·금액 차이를 원본 위치와 함께 표시 |
| 06 | 출석부 OCR 대조 | 스캔 출석부와 엑셀 근무시간 자동 비교 |
| 07 | 제출 서식 생성 | 건강보험공단·일모아·인사랑 양식 원클릭 생성 |
| 08 | 현황·마감 알림 | 집행률, 미처리 건, 신고 기한 D-7/D-3/D-Day |
| 09 | 부서 Pre-Check 포털 <sub>제안</sub> | 부서가 제출 전에 스스로 오류 확인·수정 |
| 10 | AI 행정규정 챗봇 <sub>제안</sub> | 업무 지침을 근거와 출처로 답변 (RAG) |

## 문서
| 문서 | 내용 |
| :--- | :--- |
| [아키텍처](https://github.com/YC-CapstoneDesign-5/.github/blob/main/docs/ARCHITECTURE.md) | 설계 원칙, 시스템 구성도, 파이프라인, 데이터 모델, 보안 |
| [기능 명세](https://github.com/YC-CapstoneDesign-5/.github/blob/main/docs/FEATURES.md) | 10개 기능의 목적·입출력·완료 기준 |
| [로드맵](https://github.com/YC-CapstoneDesign-5/.github/blob/main/docs/ROADMAP.md) | 단계별 구현 계획과 검증 시나리오 |
| [협업 가이드](https://github.com/YC-CapstoneDesign-5/.github/blob/main/CONTRIBUTING.md) | 브랜치·커밋·PR 규칙, 로컬 개발 환경 |

## 레포지토리
| 레포 | 설명 |
| :--- | :--- |
| [Front](https://github.com/YC-CapstoneDesign-5/Front) | 웹 프론트엔드 |
| [Back](https://github.com/YC-CapstoneDesign-5/Back) | API 서버·검증/계산 엔진·비동기 워커 |

## 팀
| 이름 | 역할 |
| :--- | :--- |
| **이규빈** | **PM** |
| **박희동** | **UI/UX** |
| **황규빈** | **Frontend** |
| **제성준** | **Frontend** |
| **김정운** | **Backend** |
| **김준승** | **QA** |
| **황원경** | **Documentation** |

<sub>기술 스택은 팀 협의 후 확정 예정입니다. 현재 후보는 [아키텍처 문서](https://github.com/YC-CapstoneDesign-5/.github/blob/main/docs/ARCHITECTURE.md#10-기술-스택-확정-전)를 참고하세요.</sub>
