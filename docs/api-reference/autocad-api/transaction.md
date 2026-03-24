# Transaction (트랜잭션) 패턴

> AutoCAD/Civil 3D 데이터베이스 작업의 핵심 패턴

## 개요

트랜잭션은 AutoCAD 데이터베이스 작업의 기본 단위입니다. 모든 객체 읽기/쓰기는 트랜잭션 내에서 수행해야 합니다.

## 기본 패턴

```csharp
using Autodesk.AutoCAD.DatabaseServices;

public void BasicTransactionPattern()
{
    Document doc = Application.DocumentManager.MdiActiveDocument;
    Database db = doc.Database;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        try
        {
            // 작업 수행
            // ...

            tr.Commit();  // 성공 시 커밋
        }
        catch (Exception ex)
        {
            // 예외 발생 시 자동 롤백 (Commit 없이 Dispose)
            doc.Editor.WriteMessage($"\n오류: {ex.Message}");
        }
    }
    // using 블록 종료 시 트랜잭션 자동 Dispose
}
```

## 객체 열기 모드

| 모드 | 용도 | 잠금 |
|------|------|------|
| OpenMode.ForRead | 읽기 전용 | 공유 잠금 |
| OpenMode.ForWrite | 읽기/쓰기 | 배타적 잠금 |
| OpenMode.ForNotify | 알림만 | 없음 |

```csharp
// 읽기 전용
Entity entity = tr.GetObject(entityId, OpenMode.ForRead) as Entity;

// 수정이 필요한 경우
Entity entity = tr.GetObject(entityId, OpenMode.ForWrite) as Entity;

// 읽기 → 쓰기로 업그레이드
entity.UpgradeOpen();
```

## 테이블 접근 패턴

### BlockTable / BlockTableRecord

```csharp
using (Transaction tr = db.TransactionManager.StartTransaction())
{
    // 블록 테이블 열기
    BlockTable bt = tr.GetObject(db.BlockTableId, OpenMode.ForRead) as BlockTable;

    // 모델 공간 열기 (쓰기 모드)
    BlockTableRecord modelSpace = tr.GetObject(
        bt[BlockTableRecord.ModelSpace],
        OpenMode.ForWrite
    ) as BlockTableRecord;

    // 새 엔티티 추가
    Line line = new Line(new Point3d(0, 0, 0), new Point3d(100, 100, 0));
    modelSpace.AppendEntity(line);
    tr.AddNewlyCreatedDBObject(line, true);

    tr.Commit();
}
```

### LayerTable

```csharp
using (Transaction tr = db.TransactionManager.StartTransaction())
{
    LayerTable lt = tr.GetObject(db.LayerTableId, OpenMode.ForRead) as LayerTable;

    foreach (ObjectId layerId in lt)
    {
        LayerTableRecord layer = tr.GetObject(layerId, OpenMode.ForRead) as LayerTableRecord;
        doc.Editor.WriteMessage($"\n도면층: {layer.Name}");
    }

    tr.Commit();
}
```

## 중첩 트랜잭션

```csharp
using (Transaction outerTr = db.TransactionManager.StartTransaction())
{
    // 외부 트랜잭션 작업

    using (Transaction innerTr = db.TransactionManager.StartTransaction())
    {
        // 내부 트랜잭션 작업
        innerTr.Commit();  // 내부 커밋
    }

    outerTr.Commit();  // 외부 커밋 (실제 DB 반영)
}
```

## Document Lock

멀티스레드 환경이나 모달리스 대화상자에서 문서 수정 시:

```csharp
public void ModifyWithDocumentLock()
{
    Document doc = Application.DocumentManager.MdiActiveDocument;

    using (DocumentLock docLock = doc.LockDocument())
    {
        using (Transaction tr = doc.Database.TransactionManager.StartTransaction())
        {
            // 안전하게 수정 작업 수행
            tr.Commit();
        }
    }
}
```

## 성능 최적화

### 다수 객체 처리

```csharp
// 비효율적 (매번 새 트랜잭션)
foreach (ObjectId id in objectIds)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Entity ent = tr.GetObject(id, OpenMode.ForRead) as Entity;
        // 처리
        tr.Commit();
    }
}

// 효율적 (단일 트랜잭션)
using (Transaction tr = db.TransactionManager.StartTransaction())
{
    foreach (ObjectId id in objectIds)
    {
        Entity ent = tr.GetObject(id, OpenMode.ForRead) as Entity;
        // 처리
    }
    tr.Commit();
}
```

### OpenCloseTransaction (대용량 처리)

```csharp
using (OpenCloseTransaction tr = new OpenCloseTransaction())
{
    foreach (ObjectId id in largeCollection)
    {
        Entity ent = tr.GetObject(id, OpenMode.ForRead) as Entity;
        // 처리 후 즉시 닫힘
    }
    tr.Commit();
}
```

## 일반적인 실수

### 1. Commit 누락

```csharp
// 잘못된 예 - 변경사항 손실
using (Transaction tr = db.TransactionManager.StartTransaction())
{
    // 수정 작업
}  // Commit 없이 종료 → 롤백됨

// 올바른 예
using (Transaction tr = db.TransactionManager.StartTransaction())
{
    // 수정 작업
    tr.Commit();  // 명시적 커밋
}
```

### 2. 트랜잭션 외부에서 객체 접근

```csharp
Entity entity;
using (Transaction tr = db.TransactionManager.StartTransaction())
{
    entity = tr.GetObject(entityId, OpenMode.ForRead) as Entity;
    tr.Commit();
}
// entity 사용 → 오류! 트랜잭션 종료 후 객체 무효

// 해결: 트랜잭션 내에서 필요한 데이터만 추출
Point3d startPoint;
using (Transaction tr = db.TransactionManager.StartTransaction())
{
    Line line = tr.GetObject(lineId, OpenMode.ForRead) as Line;
    startPoint = line.StartPoint;  // 값 복사
    tr.Commit();
}
// startPoint는 안전하게 사용 가능
```

---

*관련: [Database Access](./database-access.md) | [Entity Manipulation](./entity-manipulation.md)*
