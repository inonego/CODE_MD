---
name: csharp-conventions
description: Unity C# 코드 작성 및 수정 시 적용하는 스타일 규칙. C# 파일(.cs)을 생성하거나 수정할 때 사용. BSD 중괄호, 한국어 주석, region 구조, 네이밍 규칙 등을 포함. Use when writing, editing, or reviewing C# code files.
---

# C# 코드 스타일 규칙

Unity C# / 한국어 기반. 코드 생성 요청 시 즉시 적용.
가독성을 최우선으로 작성. 논리 단위 사이에 빈 줄을 넣어 시각적으로 구분하고, 의미가 달라지는 지점마다 주석이나 구분선으로 맥락을 전달.

## 중괄호

BSD 스타일 (Allman). 여는 중괄호를 새 줄에 배치.

## 주석

- 클래스/인터페이스: `=` 구분선 60자 + XML `<summary>`
- 메서드/프로퍼티: `-` 구분선 60자 + XML `<summary>`
- **구분선 길이 조정**: 주석 내용이 긴 경우 구분선을 10자씩 추가 가능 (60 → 70 → 80...)
- XML 문서 주석은 `<summary>`만 사용 (`<br/>` 허용)
- 순서가 있는 주석이라도 번호를 붙이지 않음 (`// 1. 초기화` X → `// 초기화` O)

```csharp
// ============================================================
/// <summary>
/// 클래스 설명
/// </summary>
// ============================================================
public class MyClass { }

// ------------------------------------------------------------
/// <summary>
/// <br/> 메서드 설명
/// <br/> 여러줄인 경우 이렇게 작성 
/// </summary>
// ------------------------------------------------------------
public void MyMethod() { }
```

## 네임스페이스

### Using 정렬

System → Unity → 라이브러리 → 프로젝트 (그룹 간 빈 줄)

### 선언

- **순차적 선언 원칙**: 하위 네임스페이스를 사용하더라도 상위 네임스페이스를 순차적으로 모두 선언
  - 예: `using System.Collections.Generic;`을 사용하는 경우 → `using System;`, `using System.Collections;`, `using System.Collections.Generic;` 순서로 작성
- 현재 네임스페이스의 하위/동등 네임스페이스는 namespace 블록 **내부**에서 using (내부에서도 순차적 선언 원칙 적용)
- 완전히 다른 네임스페이스는 파일 상단에서 using

```csharp
// Collections.Generic만 사용하더라도 System, Collections를 순차적으로 선언
using System;
using System.Collections;
using System.Collections.Generic;

using UnityEngine;

namespace inonego
{
   // namespace 내부에서도 순차적 선언 원칙 적용
   using Internal;
   using Internal.Namespace;  // inonego.Internal.Namespace → 내부이므로 namespace 안쪽
   using Other;
   using Other.Sub;           // inonego.Other.Sub → 내부이므로 namespace 안쪽
```

## Region 순서

필드 → 이벤트 → 생성자 → 복제 관련 → 메서드 → 이벤트 핸들러 → 인터페이스 구현 → 기타

- 기능별 커스텀 region 허용 (예: `#region 키 설정`, `#region 체력 관련`)
- region 레이블은 클래스 인덴트와 동일 (내부 코드는 한 단계 더 들여쓰기)

## 네이밍

| 대상 | 규칙 | 예시 |
|------|------|------|
| private 필드 | camelCase | `private int health = 0;` |
| public 필드 | PascalCase | `public int Health = 0;` |
| 메서드 | PascalCase | `public void Spawn()` |
| 프로퍼티 | PascalCase | `public int Value { get; set; }` |

## 필드/프로퍼티

- 직렬화: `[SerializeField]` 또는 `[SerializeReference]` 사용
- 선언 시 기본값 초기화
- 단순 반환/설정: Expression Body (`=> value`)
- 읽기 전용: `=>`
- 로직 필요 시 일반 get/set
- 필드 region 내 프로퍼티와 backing field 함께 배치 (private은 프로퍼티 아래)

## 생성자

- 기본 생성자: `: base()` 또는 `: this()`
- 매개변수 생성자: `: this()` 체이닝

## 메서드

- 선택적 매개변수 기본값 허용
- 단순 로직만 Expression Body
- 비동기 작업은 async/await

## 클래스/인터페이스/Enum

- MonoBehaviour가 아닌 클래스: `[Serializable]` 권장
- Enum: 클래스 내부 = 전용, 클래스 외부 = 공용
- Enum: 간단하면 한 줄, 많으면 여러 줄

## Null 체크

명시적 null 체크 사용. 예외 메시지는 한국어.

```csharp
if (obj == null)
{
   throw new ArgumentNullException("값이 null입니다.");
}
```

| 예외 타입 | 사용 시점 |
|----------|----------|
| `ArgumentNullException` | 매개변수가 null |
| `InvalidOperationException` | 해당 방식으로 수행 불가 |
| `NullReferenceException` | 필드/프로퍼티 미설정 |

## 전체 예시

전체 규칙이 적용된 클래스 예시는 [references/example.md](references/example.md) 참조.
