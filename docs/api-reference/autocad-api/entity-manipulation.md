# Entity Manipulation (엔티티 조작)

> AutoCAD 엔티티 생성, 수정, 삭제 방법

## 개요

Entity는 AutoCAD 도면에 표시되는 모든 그래픽 객체의 기본 클래스입니다.

## 엔티티 계층 구조

```
DBObject
└── Entity
    ├── Curve
    │   ├── Line
    │   ├── Arc
    │   ├── Circle
    │   ├── Polyline
    │   ├── Polyline2d
    │   ├── Polyline3d
    │   └── Spline
    ├── DBText
    ├── MText
    ├── BlockReference
    ├── Hatch
    ├── Dimension (추상)
    │   ├── AlignedDimension
    │   ├── RotatedDimension
    │   └── ...
    └── ...
```

## 기본 엔티티 생성

### Line (선)

```csharp
public void CreateLine()
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTableRecord modelSpace = GetModelSpace(tr, OpenMode.ForWrite);

        Line line = new Line(
            new Point3d(0, 0, 0),    // 시작점
            new Point3d(100, 50, 0)  // 끝점
        );

        // 속성 설정 (선택)
        line.Layer = "MyLayer";
        line.ColorIndex = 1;  // 빨강

        modelSpace.AppendEntity(line);
        tr.AddNewlyCreatedDBObject(line, true);

        tr.Commit();
    }
}
```

### Circle (원)

```csharp
public void CreateCircle()
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTableRecord modelSpace = GetModelSpace(tr, OpenMode.ForWrite);

        Circle circle = new Circle(
            new Point3d(50, 50, 0),  // 중심점
            Vector3d.ZAxis,          // 법선 벡터
            25                       // 반지름
        );

        modelSpace.AppendEntity(circle);
        tr.AddNewlyCreatedDBObject(circle, true);

        tr.Commit();
    }
}
```

### Arc (호)

```csharp
public void CreateArc()
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTableRecord modelSpace = GetModelSpace(tr, OpenMode.ForWrite);

        Arc arc = new Arc(
            new Point3d(50, 50, 0),  // 중심점
            30,                      // 반지름
            0,                       // 시작 각도 (라디안)
            Math.PI / 2              // 끝 각도 (라디안, 90도)
        );

        modelSpace.AppendEntity(arc);
        tr.AddNewlyCreatedDBObject(arc, true);

        tr.Commit();
    }
}
```

### Polyline (폴리선)

```csharp
public void CreatePolyline()
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTableRecord modelSpace = GetModelSpace(tr, OpenMode.ForWrite);

        Polyline pline = new Polyline();

        // 정점 추가 (2D 좌표, Bulge는 곡률)
        pline.AddVertexAt(0, new Point2d(0, 0), 0, 0, 0);
        pline.AddVertexAt(1, new Point2d(100, 0), 0, 0, 0);
        pline.AddVertexAt(2, new Point2d(100, 50), 0.5, 0, 0);  // Bulge=0.5 (곡선)
        pline.AddVertexAt(3, new Point2d(0, 50), 0, 0, 0);

        pline.Closed = true;  // 닫힌 폴리선

        modelSpace.AppendEntity(pline);
        tr.AddNewlyCreatedDBObject(pline, true);

        tr.Commit();
    }
}
```

### DBText (단일행 문자)

```csharp
public void CreateText()
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTableRecord modelSpace = GetModelSpace(tr, OpenMode.ForWrite);

        DBText text = new DBText();
        text.Position = new Point3d(0, 0, 0);
        text.TextString = "Hello Civil 3D";
        text.Height = 2.5;
        text.Rotation = 0;  // 라디안

        modelSpace.AppendEntity(text);
        tr.AddNewlyCreatedDBObject(text, true);

        tr.Commit();
    }
}
```

### MText (다중행 문자)

```csharp
public void CreateMText()
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTableRecord modelSpace = GetModelSpace(tr, OpenMode.ForWrite);

        MText mtext = new MText();
        mtext.Location = new Point3d(0, 0, 0);
        mtext.Contents = "첫 번째 줄\\P두 번째 줄";  // \\P = 줄바꿈
        mtext.TextHeight = 2.5;
        mtext.Width = 100;  // 텍스트 박스 폭

        modelSpace.AppendEntity(mtext);
        tr.AddNewlyCreatedDBObject(mtext, true);

        tr.Commit();
    }
}
```

### BlockReference (블록 참조)

```csharp
public void InsertBlock(string blockName, Point3d insertPoint, double scale, double rotation)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTable bt = tr.GetObject(db.BlockTableId, OpenMode.ForRead) as BlockTable;

        if (!bt.Has(blockName))
        {
            ed.WriteMessage($"\n블록 '{blockName}'을 찾을 수 없습니다.");
            return;
        }

        BlockTableRecord modelSpace = GetModelSpace(tr, OpenMode.ForWrite);

        BlockReference blockRef = new BlockReference(insertPoint, bt[blockName]);
        blockRef.ScaleFactors = new Scale3d(scale);
        blockRef.Rotation = rotation;  // 라디안

        modelSpace.AppendEntity(blockRef);
        tr.AddNewlyCreatedDBObject(blockRef, true);

        tr.Commit();
    }
}
```

## 엔티티 수정

### 속성 변경

```csharp
public void ModifyEntity(ObjectId entityId)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Entity entity = tr.GetObject(entityId, OpenMode.ForWrite) as Entity;

        if (entity != null)
        {
            // 공통 속성
            entity.Layer = "NewLayer";
            entity.ColorIndex = 3;  // 초록
            entity.Linetype = "DASHED";
            entity.LineWeight = LineWeight.LineWeight050;

            // 타입별 속성
            if (entity is Line line)
            {
                line.StartPoint = new Point3d(0, 0, 0);
                line.EndPoint = new Point3d(200, 100, 0);
            }
            else if (entity is Circle circle)
            {
                circle.Radius = 50;
            }
        }

        tr.Commit();
    }
}
```

### 이동 (Move)

```csharp
public void MoveEntity(ObjectId entityId, Vector3d displacement)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Entity entity = tr.GetObject(entityId, OpenMode.ForWrite) as Entity;

        Matrix3d moveMatrix = Matrix3d.Displacement(displacement);
        entity.TransformBy(moveMatrix);

        tr.Commit();
    }
}
```

### 회전 (Rotate)

```csharp
public void RotateEntity(ObjectId entityId, Point3d basePoint, double angle)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Entity entity = tr.GetObject(entityId, OpenMode.ForWrite) as Entity;

        Matrix3d rotateMatrix = Matrix3d.Rotation(angle, Vector3d.ZAxis, basePoint);
        entity.TransformBy(rotateMatrix);

        tr.Commit();
    }
}
```

### 축척 (Scale)

```csharp
public void ScaleEntity(ObjectId entityId, Point3d basePoint, double scaleFactor)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Entity entity = tr.GetObject(entityId, OpenMode.ForWrite) as Entity;

        Matrix3d scaleMatrix = Matrix3d.Scaling(scaleFactor, basePoint);
        entity.TransformBy(scaleMatrix);

        tr.Commit();
    }
}
```

### 복사 (Copy)

```csharp
public ObjectId CopyEntity(ObjectId entityId, Vector3d displacement)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Entity entity = tr.GetObject(entityId, OpenMode.ForRead) as Entity;
        Entity clone = entity.Clone() as Entity;

        Matrix3d moveMatrix = Matrix3d.Displacement(displacement);
        clone.TransformBy(moveMatrix);

        BlockTableRecord modelSpace = GetModelSpace(tr, OpenMode.ForWrite);
        ObjectId newId = modelSpace.AppendEntity(clone);
        tr.AddNewlyCreatedDBObject(clone, true);

        tr.Commit();
        return newId;
    }
}
```

## 엔티티 삭제

```csharp
public void EraseEntity(ObjectId entityId)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Entity entity = tr.GetObject(entityId, OpenMode.ForWrite) as Entity;
        entity.Erase();

        tr.Commit();
    }
}

// 여러 엔티티 삭제
public void EraseEntities(ObjectIdCollection entityIds)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        foreach (ObjectId id in entityIds)
        {
            Entity entity = tr.GetObject(id, OpenMode.ForWrite) as Entity;
            entity.Erase();
        }

        tr.Commit();
    }
}
```

## 유틸리티 메서드

### 모델 공간 가져오기

```csharp
private BlockTableRecord GetModelSpace(Transaction tr, OpenMode mode)
{
    BlockTable bt = tr.GetObject(db.BlockTableId, OpenMode.ForRead) as BlockTable;
    return tr.GetObject(bt[BlockTableRecord.ModelSpace], mode) as BlockTableRecord;
}
```

### 엔티티 정보 조회

```csharp
public void GetEntityInfo(ObjectId entityId)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Entity entity = tr.GetObject(entityId, OpenMode.ForRead) as Entity;

        ed.WriteMessage($"\n타입: {entity.GetType().Name}");
        ed.WriteMessage($"\n도면층: {entity.Layer}");
        ed.WriteMessage($"\n색상: {entity.Color}");
        ed.WriteMessage($"\n선종류: {entity.Linetype}");

        // 기하학적 범위
        Extents3d? extents = entity.Bounds;
        if (extents.HasValue)
        {
            ed.WriteMessage($"\n범위: {extents.Value.MinPoint} ~ {extents.Value.MaxPoint}");
        }

        tr.Commit();
    }
}
```

## 주의사항

- 새 엔티티 생성 후 `AppendEntity()` + `AddNewlyCreatedDBObject()` 필수
- 수정 시 `OpenMode.ForWrite`로 열기
- `Erase()`는 즉시 삭제가 아닌 삭제 표시 (Commit 시 실제 삭제)
- 변환(Transform)은 Matrix3d 사용

---

*관련: [Database Access](./database-access.md) | [Selection](./selection.md) | [Geometry](./geometry.md)*
