---
name: csharp-conventions
description: C# 파일 작성/수정 필수 로드. 읽기 전용 작업 제외. Unity C# 스타일 규칙
user-invocable: false
---

# C# 코드 스타일 규칙

Unity C#. 코드 생성 요청 시 즉시 적용.
가독성을 최우선으로 작성. 논리 단위 사이에 빈 줄을 넣어 시각적으로 구분하고, 의미가 달라지는 지점마다 주석이나 구분선으로 맥락을 전달.

```csharp
// [using 정렬] System → Unity → 라이브러리 → 프로젝트 (그룹 간 빈 줄)
// [순차적 선언] 하위만 쓰더라도 상위를 모두 선언
using System;
using System.Collections;
using System.Collections.Generic; // Collections.Generic만 써도 위 두 줄 필수
using System.Linq;

using UnityEngine;

namespace inonego
{
   // 내부에서도 순차적 선언 원칙 적용
   // 같은/하위 네임스페이스는 namespace 블록 내부에서 상대 이름으로 선언
   // ex) inonego.Internal이 아닌 Internal로 작성 (부모 네임스페이스 생략)
   using Internal;
   using Internal.Data;

   // [주석: 클래스/인터페이스] = 구분선 60자 + XML <summary>
   // ============================================================
   /// <summary>
   /// 인터페이스 설명
   /// </summary>
   // ============================================================
   public interface IExampleValue
   {

   }

   // ============================================================
   /// <summary>
   /// 클래스 설명
   /// </summary>
   // ============================================================
   // [클래스] MonoBehaviour 아닌 클래스는 [Serializable] 권장
   [Serializable]
   public abstract class Example<TKey, T>
   where TKey : IEquatable<TKey>
   where T : class, IExampleValue
   // 제네릭 제약 조건은 클래스 선언과 같은 인덴트
   {
   // [중괄호] BSD 스타일 (Allman) — 여는 중괄호를 새 줄에 배치

   // [region 순서] 필드 → 이벤트 → 생성자 → 복제 → 메서드 → 이벤트 핸들러 → 인터페이스 구현 → 기타
   // [region 인덴트] 레이블은 클래스 인덴트, 내부 코드는 한 단계 더 들여쓰기
   // [region 생략] 내용이 없는 region은 선언하지 않음 (예: 필드가 없으면 #region 필드 자체를 작성하지 않음)
   // 기능별 커스텀 region 허용 (예: #region 키 설정)

   #region 내부 데이터

      [Serializable]
      public struct Point
      {
         // [네이밍] public 필드 = PascalCase
         public int X;
         public int Y;
      }

   #endregion

   #region 필드

      // [주석: 메서드/프로퍼티] - 구분선 60자 + XML <summary>
      // ------------------------------------------------------------
      /// <summary>
      /// 프로퍼티 설명
      /// </summary>
      // ------------------------------------------------------------
      // [프로퍼티] 로직 필요 시 일반 get/set
      public GameObject Value
      {
         // [프로퍼티] 단순 반환 = Expression Body
         get => value;
         set
         {
            var (prev, next) = (this.value, value);

            if (prev == next) return;

            if (prev != null)
            {
               // prev에 대한 해제 작업
            }

            this.value = next;

            if (next != null)
            {
               // next에 대한 초기화 작업
            }

            OnValueChange?.Invoke(this, new ValueChangeEventArgs { Value = 0 });
         }
      }

      // [필드] 직렬화는 [SerializeField] 사용, 기본값 초기화
      // [필드 배치] 프로퍼티와 backing field 함께 배치 (private은 프로퍼티 아래)
      [SerializeField]
      private GameObject value = null; // [네이밍] private 필드 = camelCase

      // ------------------------------------------------------------
      /// <summary>
      /// 읽기 전용 프로퍼티 설명
      /// </summary>
      // ------------------------------------------------------------
      // [프로퍼티] 읽기 전용 = =>
      public bool IsActive => isActive;

      [SerializeField]
      private bool isActive = false;

      // SerializeField가 없으면 위의 public 프로퍼티와 붙여쓰기

   #endregion

   #region 이벤트

      // [Enum/구조체] 클래스 내부 = 전용, 외부 = 공용
      // [Enum] 간단하면 한 줄, 많으면 여러 줄
      [Serializable]
      public struct ValueChangeEventArgs
      {
         public int Value;
      }

      // ------------------------------------------------------------
      /// <summary>
      /// 이벤트 설명
      /// </summary>
      // ------------------------------------------------------------
      public event EventHandler<ValueChangeEventArgs> OnValueChange = null;

   #endregion

   #region 생성자

      // [생성자] 기본 생성자: base() 또는 this()
      public Example() : base() {}

      // [생성자] 매개변수 생성자: this() 체이닝
      public Example(GameObject value) : this()
      {
         // [제어문] 단일 문장이라도 반드시 중괄호
         if (value == null)
         {
            throw new ArgumentNullException("값이 null입니다.");
         }

         this.value = value;
      }

   #endregion

   #region 메서드

      // ------------------------------------------------------------
      /// <summary>
      /// 메서드 설명
      /// </summary>
      // ------------------------------------------------------------
      // [네이밍] 메서드 = PascalCase
      // [메서드] 선택적 매개변수 기본값 허용, 단순 로직만 Expression Body, 비동기는 async/await
      public void Spawn()
      {
         if (value == null)
         {
            throw new InvalidOperationException("값이 설정되어 있지 않습니다.");
         }

         var spawnable = Instantiate(value);
         var hasError = false;

         if (hasError)
         {
            DespawnInternal(spawnable);
         }

         // [주석] 순서가 있더라도 번호 금지 (// 1. X → // 초기화 O)
         // [주석 변경 이력 금지] 기존 주석에 (변경 내용) 괄호 설명 덧붙이기 금지
         // 기능이 바뀌어 주석이 틀려진 경우에만 현재 상태로 수정.
         OnBeforeSpawn(spawnable);
         spawnable.SetActive(true);

         OnSpawnComplete?.Invoke(this, spawnable);
      }

      // [주석: 구분선 길이 조정] 구분선은 내부 내용(/// 줄)의 시각적 폭보다 길어야 함
      // 기본 60자, 내용이 길면 10자씩 추가 (60 → 70 → 80 → ...)
      // 한글은 모노스페이스에서 2칸 차지하므로 시각적 폭 기준으로 판단
      // ------------------------------------------------------------
      /// <summary>
      /// <br/> 주석 내용이 긴 경우 구분선을 10자씩 추가한 예시
      /// <br/> 여러줄인 경우 매 줄 앞에 br 사용
      /// </summary>
      // ------------------------------------------------------------
      public void ComplexMethodWithLongDescription()
      {
         // [제어문] 단일 문장이라도 반드시 중괄호, 한 줄 작성 금지
         var list = new List<int> { 1, 2, 3 };

         // for (int i = 0; i < list.Count; i++) Debug.Log(list[i]); ← 금지
         for (int i = 0; i < list.Count; i++)
         {
            Debug.Log(list[i]);
         }

         // [호출] 긴 변수명은 짧은 지역 변수로 할당 (의미 유지. p X → position O)
         var position = spawnPointObject.transform.position;
         var rotation = spawnPointObject.transform.rotation;
         var scale    = spawnPointObject.transform.localScale;

         // [호출] 여러 줄이 되면 괄호를 중괄호처럼 사용
         LongParameterMethod
         (
            position, rotation, scale,
            "Example Name", true, 100
         );
      }

      // [선언부] 호출과 동일한 규칙 적용
      private void LongParameterMethod
      (
         Vector3 position, Quaternion rotation, Vector3 scale,
         string name, bool isActive, int count
      )
      {
         // 구현 내용
      }

   #endregion

   }
}
```
