# Selection (선택 및 입력)

> 사용자 입력 처리 및 객체 선택 방법

## 개요

Editor 클래스를 통해 사용자로부터 점, 문자열, 숫자, 객체 선택 등 다양한 입력을 받을 수 있습니다.

## Editor 가져오기

```csharp
Document doc = Application.DocumentManager.MdiActiveDocument;
Editor ed = doc.Editor;
```

## 점 입력

### 단일 점 선택

```csharp
public Point3d? GetPoint(string prompt)
{
    PromptPointOptions ppo = new PromptPointOptions(prompt);
    ppo.AllowNone = false;  // 빈 입력 불허

    PromptPointResult ppr = ed.GetPoint(ppo);

    if (ppr.Status == PromptStatus.OK)
    {
        return ppr.Value;
    }

    return null;
}

// 사용
Point3d? point = GetPoint("\n점을 선택하세요: ");
if (point.HasValue)
{
    ed.WriteMessage($"\n선택된 점: {point.Value}");
}
```

### 기준점 기반 점 선택

```csharp
public Point3d? GetPointWithBase(string prompt, Point3d basePoint)
{
    PromptPointOptions ppo = new PromptPointOptions(prompt);
    ppo.BasePoint = basePoint;
    ppo.UseBasePoint = true;  // 기준점에서 러버밴드 라인 표시

    PromptPointResult ppr = ed.GetPoint(ppo);

    return ppr.Status == PromptStatus.OK ? ppr.Value : (Point3d?)null;
}
```

## 거리/각도 입력

### 거리 입력

```csharp
public double? GetDistance(string prompt)
{
    PromptDistanceOptions pdo = new PromptDistanceOptions(prompt);
    pdo.AllowNegative = false;
    pdo.AllowZero = false;
    pdo.DefaultValue = 10.0;
    pdo.UseDefaultValue = true;

    PromptDoubleResult pdr = ed.GetDistance(pdo);

    return pdr.Status == PromptStatus.OK ? pdr.Value : (double?)null;
}
```

### 각도 입력

```csharp
public double? GetAngle(string prompt, Point3d basePoint)
{
    PromptAngleOptions pao = new PromptAngleOptions(prompt);
    pao.BasePoint = basePoint;
    pao.UseBasePoint = true;
    pao.DefaultValue = 0;

    PromptDoubleResult pdr = ed.GetAngle(pao);

    return pdr.Status == PromptStatus.OK ? pdr.Value : (double?)null;
}
```

## 문자열/숫자 입력

### 문자열 입력

```csharp
public string GetString(string prompt, string defaultValue = "")
{
    PromptStringOptions pso = new PromptStringOptions(prompt);
    pso.AllowSpaces = true;  // 공백 허용
    pso.DefaultValue = defaultValue;
    pso.UseDefaultValue = !string.IsNullOrEmpty(defaultValue);

    PromptResult pr = ed.GetString(pso);

    return pr.Status == PromptStatus.OK ? pr.StringResult : null;
}
```

### 정수 입력

```csharp
public int? GetInteger(string prompt, int defaultValue = 0)
{
    PromptIntegerOptions pio = new PromptIntegerOptions(prompt);
    pio.AllowNegative = false;
    pio.AllowZero = true;
    pio.DefaultValue = defaultValue;
    pio.UseDefaultValue = true;

    PromptIntegerResult pir = ed.GetInteger(pio);

    return pir.Status == PromptStatus.OK ? pir.Value : (int?)null;
}
```

### 실수 입력

```csharp
public double? GetDouble(string prompt, double defaultValue = 0)
{
    PromptDoubleOptions pdo = new PromptDoubleOptions(prompt);
    pdo.AllowNegative = true;
    pdo.AllowZero = true;
    pdo.DefaultValue = defaultValue;
    pdo.UseDefaultValue = true;

    PromptDoubleResult pdr = ed.GetDouble(pdo);

    return pdr.Status == PromptStatus.OK ? pdr.Value : (double?)null;
}
```

## 키워드 선택

```csharp
public string GetKeyword(string prompt, string[] keywords, string defaultKeyword = null)
{
    PromptKeywordOptions pko = new PromptKeywordOptions(prompt);

    foreach (string kw in keywords)
    {
        pko.Keywords.Add(kw);
    }

    if (!string.IsNullOrEmpty(defaultKeyword))
    {
        pko.Keywords.Default = defaultKeyword;
    }

    pko.AllowNone = false;

    PromptResult pr = ed.GetKeywords(pko);

    return pr.Status == PromptStatus.OK ? pr.StringResult : null;
}

// 사용 예
string choice = GetKeyword(
    "\n옵션 선택 [예(Y)/아니오(N)]",
    new[] { "Y", "N" },
    "Y"
);
```

## 객체 선택

### 단일 객체 선택

```csharp
public ObjectId? SelectSingleEntity(string prompt)
{
    PromptEntityOptions peo = new PromptEntityOptions(prompt);
    peo.AllowNone = false;

    PromptEntityResult per = ed.GetEntity(peo);

    return per.Status == PromptStatus.OK ? per.ObjectId : (ObjectId?)null;
}
```

### 특정 타입만 선택

```csharp
public ObjectId? SelectEntityOfType<T>(string prompt) where T : Entity
{
    PromptEntityOptions peo = new PromptEntityOptions(prompt);
    peo.SetRejectMessage($"\n{typeof(T).Name} 타입만 선택 가능합니다.");
    peo.AddAllowedClass(typeof(T), true);

    PromptEntityResult per = ed.GetEntity(peo);

    return per.Status == PromptStatus.OK ? per.ObjectId : (ObjectId?)null;
}

// 사용
ObjectId? lineId = SelectEntityOfType<Line>("\n선을 선택하세요: ");
ObjectId? circleId = SelectEntityOfType<Circle>("\n원을 선택하세요: ");
```

### 다중 객체 선택 (Selection Set)

```csharp
public ObjectIdCollection SelectEntities(string prompt)
{
    PromptSelectionOptions pso = new PromptSelectionOptions();
    pso.MessageForAdding = prompt;

    PromptSelectionResult psr = ed.GetSelection(pso);

    if (psr.Status == PromptStatus.OK)
    {
        SelectionSet ss = psr.Value;
        return new ObjectIdCollection(ss.GetObjectIds());
    }

    return new ObjectIdCollection();
}
```

### 필터를 사용한 선택

```csharp
public ObjectIdCollection SelectByFilter(string prompt, SelectionFilter filter)
{
    PromptSelectionOptions pso = new PromptSelectionOptions();
    pso.MessageForAdding = prompt;

    PromptSelectionResult psr = ed.GetSelection(pso, filter);

    if (psr.Status == PromptStatus.OK)
    {
        return new ObjectIdCollection(psr.Value.GetObjectIds());
    }

    return new ObjectIdCollection();
}

// 사용 예: 특정 도면층의 선만 선택
public ObjectIdCollection SelectLinesOnLayer(string layerName)
{
    TypedValue[] filterList = new TypedValue[]
    {
        new TypedValue((int)DxfCode.Start, "LINE"),           // 엔티티 타입
        new TypedValue((int)DxfCode.LayerName, layerName)     // 도면층
    };

    SelectionFilter filter = new SelectionFilter(filterList);

    return SelectByFilter("\n선을 선택하세요: ", filter);
}
```

### 영역으로 선택

```csharp
// 윈도우 선택 (완전히 포함된 객체만)
public ObjectIdCollection SelectByWindow(Point3d corner1, Point3d corner2)
{
    PromptSelectionResult psr = ed.SelectWindow(corner1, corner2);

    if (psr.Status == PromptStatus.OK)
    {
        return new ObjectIdCollection(psr.Value.GetObjectIds());
    }

    return new ObjectIdCollection();
}

// 크로싱 선택 (걸치는 객체도 포함)
public ObjectIdCollection SelectByCrossing(Point3d corner1, Point3d corner2)
{
    PromptSelectionResult psr = ed.SelectCrossingWindow(corner1, corner2);

    if (psr.Status == PromptStatus.OK)
    {
        return new ObjectIdCollection(psr.Value.GetObjectIds());
    }

    return new ObjectIdCollection();
}
```

### 전체 선택

```csharp
public ObjectIdCollection SelectAll(SelectionFilter filter = null)
{
    PromptSelectionResult psr;

    if (filter != null)
    {
        psr = ed.SelectAll(filter);
    }
    else
    {
        psr = ed.SelectAll();
    }

    if (psr.Status == PromptStatus.OK)
    {
        return new ObjectIdCollection(psr.Value.GetObjectIds());
    }

    return new ObjectIdCollection();
}
```

## SelectionFilter DXF 코드

| DxfCode | 값 | 설명 |
|---------|-----|------|
| Start | 0 | 엔티티 타입 ("LINE", "CIRCLE", etc.) |
| LayerName | 8 | 도면층 이름 |
| Color | 62 | 색상 인덱스 |
| Linetype | 6 | 선종류 이름 |

### 복합 필터 예제

```csharp
// 빨간색 선 또는 파란색 원 선택
TypedValue[] filterList = new TypedValue[]
{
    new TypedValue((int)DxfCode.Operator, "<OR"),
        new TypedValue((int)DxfCode.Operator, "<AND"),
            new TypedValue((int)DxfCode.Start, "LINE"),
            new TypedValue((int)DxfCode.Color, 1),  // 빨강
        new TypedValue((int)DxfCode.Operator, "AND>"),
        new TypedValue((int)DxfCode.Operator, "<AND"),
            new TypedValue((int)DxfCode.Start, "CIRCLE"),
            new TypedValue((int)DxfCode.Color, 5),  // 파랑
        new TypedValue((int)DxfCode.Operator, "AND>"),
    new TypedValue((int)DxfCode.Operator, "OR>")
};

SelectionFilter filter = new SelectionFilter(filterList);
```

## PromptStatus 처리

```csharp
public void HandlePromptStatus(PromptResult result)
{
    switch (result.Status)
    {
        case PromptStatus.OK:
            // 정상 입력
            break;
        case PromptStatus.Cancel:
            // ESC 또는 취소
            ed.WriteMessage("\n취소되었습니다.");
            break;
        case PromptStatus.None:
            // 빈 입력 (Enter)
            break;
        case PromptStatus.Keyword:
            // 키워드 입력됨
            break;
        case PromptStatus.Error:
            // 오류 발생
            ed.WriteMessage("\n입력 오류");
            break;
    }
}
```

---

*관련: [Entity Manipulation](./entity-manipulation.md) | [Geometry](./geometry.md)*
