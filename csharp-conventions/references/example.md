# C# 코드 스타일 전체 예시

전체 규칙을 적용한 제네릭 추상 클래스 예시.

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;

using UnityEngine;

namespace inonego
{
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
   [Serializable]
   public abstract class Example<TKey, T>
   where TKey : IEquatable<TKey>
   where T : class, IExampleValue
   // 제네릭 제약 조건은 여러 줄에 걸쳐 작성 가능하며, 클래스와 선언과 같은 인덴트를 유지한다.
   {

   #region 내부 데이터

      [Serializable]
      public struct Point
      {
         public int X;
         public int Y;
      }

   #endregion

   #region 필드

      // ------------------------------------------------------------
      /// <summary>
      /// 프로퍼티 설명
      /// </summary>
      // ------------------------------------------------------------
      public GameObject Value
      {
         get => value;
         set => value
         {
            // 필드 변경 시 이벤트 연동 및 초기화/해제 패턴
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

            OnValueChange(new () { Value = next });
         }
      }

      [SerializeField]
      private GameObject value = null;

      // ------------------------------------------------------------
      /// <summary>
      /// 읽기 전용 프로퍼티 설명
      /// </summary>
      // ------------------------------------------------------------
      public bool IsActive => isActive;

      [SerializeField]
      private bool isActive = false;

   #endregion

   #region 이벤트

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

      public Example() : base() {}

      public Example(GameObject value) : this()
      {
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
      public void Spawn()
      {
         if (value == null)
         {
            throw new InvalidOperationException("값이 설정되어 있지 않습니다.");
         }

         var spawnable = // ...

         if (condition)
         {
            // 스폰 중에 예외가 발생하면 객체를 디스폰합니다.
            DespawnInternal(spawnable);
         }

         // ------------------------------------------------------------
         /// 스폰 처리
         // ------------------------------------------------------------
         OnBeforeSpawn(spawnable);
         spawnable.OnBeforeSpawn();

         // ------------------------------------------------------------
         /// 스폰 이벤트를 호출합니다.
         // ------------------------------------------------------------
         OnSpawn?.Invoke(key, spawnable);
      }

   #endregion

   }
}
```
