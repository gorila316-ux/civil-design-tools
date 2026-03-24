# Database Access (데이터베이스 접근)

> AutoCAD 도면 데이터베이스의 테이블 접근 방법

## 개요

AutoCAD 도면은 Database 객체로 표현되며, 다양한 심볼 테이블을 포함합니다.

## 데이터베이스 구조

```
Database
├── BlockTable          → BlockTableRecord (블록 정의, ModelSpace, PaperSpace)
├── LayerTable          → LayerTableRecord (도면층)
├── LinetypeTable       → LinetypeTableRecord (선종류)
├── TextStyleTable      → TextStyleTableRecord (문자 스타일)
├── DimStyleTable       → DimStyleTableRecord (치수 스타일)
├── ViewTable           → ViewTableRecord (명명된 뷰)
├── ViewportTable       → ViewportTableRecord (뷰포트)
├── UcsTable            → UcsTableRecord (UCS)
└── RegAppTable         → RegAppTableRecord (등록된 응용프로그램)
```

## Database 가져오기

```csharp
// 현재 활성 문서의 Database
Document doc = Application.DocumentManager.MdiActiveDocument;
Database db = doc.Database;

// 또는 직접 접근
Database db = HostApplicationServices.WorkingDatabase;
```

## BlockTable / BlockTableRecord

### 모델 공간 접근

```csharp
using (Transaction tr = db.TransactionManager.StartTransaction())
{
    // BlockTable 열기
    BlockTable bt = tr.GetObject(db.BlockTableId, OpenMode.ForRead) as BlockTable;

    // 모델 공간 열기 (쓰기 모드 - 객체 추가 시)
    BlockTableRecord modelSpace = tr.GetObject(
        bt[BlockTableRecord.ModelSpace],
        OpenMode.ForWrite
    ) as BlockTableRecord;

    // 모델 공간에 엔티티 추가
    Line line = new Line(new Point3d(0, 0, 0), new Point3d(100, 100, 0));
    modelSpace.AppendEntity(line);
    tr.AddNewlyCreatedDBObject(line, true);

    tr.Commit();
}
```

### 블록 정의 조회

```csharp
public void ListAllBlocks()
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTable bt = tr.GetObject(db.BlockTableId, OpenMode.ForRead) as BlockTable;

        foreach (ObjectId blockId in bt)
        {
            BlockTableRecord btr = tr.GetObject(blockId, OpenMode.ForRead) as BlockTableRecord;

            // 익명 블록, 레이아웃 제외
            if (!btr.IsAnonymous && !btr.IsLayout)
            {
                ed.WriteMessage($"\n블록: {btr.Name}");
            }
        }

        tr.Commit();
    }
}
```

### 블록 정의 생성

```csharp
public ObjectId CreateBlockDefinition(string blockName)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTable bt = tr.GetObject(db.BlockTableId, OpenMode.ForWrite) as BlockTable;

        // 이미 존재하는지 확인
        if (bt.Has(blockName))
        {
            return bt[blockName];
        }

        // 새 블록 정의 생성
        BlockTableRecord newBlock = new BlockTableRecord();
        newBlock.Name = blockName;

        // 블록에 포함될 엔티티 추가
        Circle circle = new Circle(Point3d.Origin, Vector3d.ZAxis, 10);
        newBlock.AppendEntity(circle);

        // BlockTable에 추가
        ObjectId blockId = bt.Add(newBlock);
        tr.AddNewlyCreatedDBObject(newBlock, true);

        tr.Commit();
        return blockId;
    }
}
```

## LayerTable / LayerTableRecord

### 모든 도면층 조회

```csharp
public void ListAllLayers()
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        LayerTable lt = tr.GetObject(db.LayerTableId, OpenMode.ForRead) as LayerTable;

        foreach (ObjectId layerId in lt)
        {
            LayerTableRecord layer = tr.GetObject(layerId, OpenMode.ForRead) as LayerTableRecord;

            ed.WriteMessage($"\n도면층: {layer.Name}");
            ed.WriteMessage($"  - 색상: {layer.Color}");
            ed.WriteMessage($"  - 켜짐: {!layer.IsOff}");
            ed.WriteMessage($"  - 동결: {layer.IsFrozen}");
        }

        tr.Commit();
    }
}
```

### 도면층 생성

```csharp
public ObjectId CreateLayer(string layerName, short colorIndex)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        LayerTable lt = tr.GetObject(db.LayerTableId, OpenMode.ForWrite) as LayerTable;

        // 이미 존재하면 해당 ID 반환
        if (lt.Has(layerName))
        {
            return lt[layerName];
        }

        // 새 도면층 생성
        LayerTableRecord newLayer = new LayerTableRecord();
        newLayer.Name = layerName;
        newLayer.Color = Color.FromColorIndex(ColorMethod.ByAci, colorIndex);

        ObjectId layerId = lt.Add(newLayer);
        tr.AddNewlyCreatedDBObject(newLayer, true);

        tr.Commit();
        return layerId;
    }
}
```

### 현재 도면층 설정

```csharp
public void SetCurrentLayer(string layerName)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        LayerTable lt = tr.GetObject(db.LayerTableId, OpenMode.ForRead) as LayerTable;

        if (lt.Has(layerName))
        {
            db.Clayer = lt[layerName];
        }

        tr.Commit();
    }
}
```

## LinetypeTable

### 선종류 로드

```csharp
public void LoadLinetype(string linetypeName)
{
    // 표준 선종류 파일에서 로드
    string ltFile = HostApplicationServices.Current.FindFile(
        "acad.lin",
        db,
        FindFileHint.Default
    );

    db.LoadLineTypeFile(linetypeName, ltFile);
}
```

## TextStyleTable

### 문자 스타일 생성

```csharp
public ObjectId CreateTextStyle(string styleName, string fontName)
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        TextStyleTable tst = tr.GetObject(db.TextStyleTableId, OpenMode.ForWrite) as TextStyleTable;

        if (tst.Has(styleName))
        {
            return tst[styleName];
        }

        TextStyleTableRecord newStyle = new TextStyleTableRecord();
        newStyle.Name = styleName;
        newStyle.FileName = fontName;  // "romans.shx" 또는 "굴림"

        ObjectId styleId = tst.Add(newStyle);
        tr.AddNewlyCreatedDBObject(newStyle, true);

        tr.Commit();
        return styleId;
    }
}
```

## 테이블 존재 여부 확인 패턴

```csharp
// 범용 패턴
public bool TableHasRecord<T>(ObjectId tableId, string recordName) where T : SymbolTable
{
    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        T table = tr.GetObject(tableId, OpenMode.ForRead) as T;
        bool exists = table.Has(recordName);
        tr.Commit();
        return exists;
    }
}

// 사용 예
bool hasLayer = TableHasRecord<LayerTable>(db.LayerTableId, "MyLayer");
bool hasBlock = TableHasRecord<BlockTable>(db.BlockTableId, "MyBlock");
```

## 주요 Database 속성

| 속성 | 타입 | 설명 |
|------|------|------|
| BlockTableId | ObjectId | 블록 테이블 |
| LayerTableId | ObjectId | 도면층 테이블 |
| LinetypeTableId | ObjectId | 선종류 테이블 |
| TextStyleTableId | ObjectId | 문자 스타일 테이블 |
| DimStyleTableId | ObjectId | 치수 스타일 테이블 |
| Clayer | ObjectId | 현재 도면층 |
| Celtype | ObjectId | 현재 선종류 |
| Textstyle | ObjectId | 현재 문자 스타일 |

## 주의사항

- 테이블 수정 시 `OpenMode.ForWrite` 필수
- 새 레코드 추가 후 `AddNewlyCreatedDBObject()` 호출 필수
- 시스템 레코드(0 도면층, Continuous 선종류 등)는 삭제 불가

---

*관련: [Transaction](./transaction.md) | [Entity Manipulation](./entity-manipulation.md)*
