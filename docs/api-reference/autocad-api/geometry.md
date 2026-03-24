# Geometry (기하학)

> 좌표, 벡터, 변환 행렬 활용

## 개요

AutoCAD Geometry 네임스페이스는 3D 좌표계 기반의 기하학적 연산을 제공합니다.

```csharp
using Autodesk.AutoCAD.Geometry;
```

## Point3d (3D 점)

### 생성 및 기본 연산

```csharp
// 생성
Point3d p1 = new Point3d(10, 20, 0);
Point3d p2 = new Point3d(50, 60, 0);
Point3d origin = Point3d.Origin;  // (0, 0, 0)

// 좌표 접근
double x = p1.X;
double y = p1.Y;
double z = p1.Z;

// 배열로 변환
double[] coords = p1.ToArray();  // [10, 20, 0]

// 두 점 사이 거리
double distance = p1.DistanceTo(p2);

// 점 + 벡터 = 새 점
Vector3d v = new Vector3d(10, 0, 0);
Point3d p3 = p1 + v;  // (20, 20, 0)

// 점 - 점 = 벡터
Vector3d diff = p2 - p1;  // (40, 40, 0)
```

### Point2d

```csharp
// 2D 점 (Z 좌표 없음)
Point2d pt2d = new Point2d(10, 20);

// Point3d ↔ Point2d 변환
Point3d pt3d = new Point3d(pt2d.X, pt2d.Y, 0);
Point2d back = new Point2d(pt3d.X, pt3d.Y);
```

## Vector3d (3D 벡터)

### 생성 및 기본 연산

```csharp
// 생성
Vector3d v1 = new Vector3d(1, 0, 0);
Vector3d v2 = new Vector3d(0, 1, 0);

// 표준 단위 벡터
Vector3d xAxis = Vector3d.XAxis;  // (1, 0, 0)
Vector3d yAxis = Vector3d.YAxis;  // (0, 1, 0)
Vector3d zAxis = Vector3d.ZAxis;  // (0, 0, 1)

// 두 점으로 벡터 생성
Point3d from = new Point3d(0, 0, 0);
Point3d to = new Point3d(10, 20, 0);
Vector3d direction = to - from;  // (10, 20, 0)

// 벡터 길이
double length = v1.Length;

// 단위 벡터 (정규화)
Vector3d normalized = direction.GetNormal();

// 벡터 스칼라 곱
Vector3d scaled = v1 * 5;  // (5, 0, 0)
```

### 벡터 연산

```csharp
// 내적 (Dot Product)
double dot = v1.DotProduct(v2);  // 0 (직교)

// 외적 (Cross Product)
Vector3d cross = v1.CrossProduct(v2);  // (0, 0, 1) = ZAxis

// 두 벡터 사이 각도 (라디안)
double angle = v1.GetAngleTo(v2);  // π/2 (90도)

// 특정 평면에서의 각도
double angleOnPlane = v1.GetAngleTo(v2, Vector3d.ZAxis);
```

### 유용한 벡터 함수

```csharp
// 두 점 사이 방향 벡터 (단위 벡터)
public Vector3d GetDirection(Point3d from, Point3d to)
{
    Vector3d v = to - from;
    return v.GetNormal();
}

// 벡터 회전 (Z축 기준)
public Vector3d RotateVector(Vector3d v, double angleRadians)
{
    Matrix3d rotation = Matrix3d.Rotation(angleRadians, Vector3d.ZAxis, Point3d.Origin);
    return v.TransformBy(rotation);
}

// 수직 벡터 (2D에서)
public Vector3d GetPerpendicular(Vector3d v)
{
    return v.CrossProduct(Vector3d.ZAxis).GetNormal();
}
```

## Matrix3d (변환 행렬)

### 이동 변환

```csharp
// 이동 벡터로 행렬 생성
Vector3d displacement = new Vector3d(100, 50, 0);
Matrix3d moveMatrix = Matrix3d.Displacement(displacement);

// 점에 적용
Point3d original = new Point3d(0, 0, 0);
Point3d moved = original.TransformBy(moveMatrix);  // (100, 50, 0)
```

### 회전 변환

```csharp
// Z축 기준 90도 회전
double angle = Math.PI / 2;  // 90도 (라디안)
Point3d center = new Point3d(0, 0, 0);
Matrix3d rotateMatrix = Matrix3d.Rotation(angle, Vector3d.ZAxis, center);

// 점에 적용
Point3d rotated = new Point3d(10, 0, 0).TransformBy(rotateMatrix);  // (0, 10, 0)
```

### 축척 변환

```csharp
// 균등 축척
double scaleFactor = 2.0;
Point3d basePoint = Point3d.Origin;
Matrix3d scaleMatrix = Matrix3d.Scaling(scaleFactor, basePoint);

// 비균등 축척
Matrix3d nonUniformScale = Matrix3d.Scaling(
    new Scale3d(2, 1, 1),  // X만 2배
    basePoint
);
```

### 대칭 변환

```csharp
// 점 대칭
Point3d mirrorPoint = new Point3d(50, 0, 0);
Matrix3d pointMirror = Matrix3d.Mirroring(mirrorPoint);

// 선 대칭
Line3d mirrorLine = new Line3d(Point3d.Origin, new Point3d(0, 100, 0));  // Y축
Matrix3d lineMirror = Matrix3d.Mirroring(mirrorLine);

// 평면 대칭
Plane3d mirrorPlane = new Plane3d(Point3d.Origin, Vector3d.ZAxis);
Matrix3d planeMirror = Matrix3d.Mirroring(mirrorPlane);
```

### 변환 조합

```csharp
// 여러 변환을 순서대로 적용
Matrix3d move = Matrix3d.Displacement(new Vector3d(100, 0, 0));
Matrix3d rotate = Matrix3d.Rotation(Math.PI / 4, Vector3d.ZAxis, Point3d.Origin);
Matrix3d scale = Matrix3d.Scaling(2, Point3d.Origin);

// 조합 (오른쪽부터 적용: scale → rotate → move)
Matrix3d combined = move * rotate * scale;

// 역변환
Matrix3d inverse = combined.Inverse();
```

## 실용 예제

### 선의 중점 계산

```csharp
public Point3d GetMidPoint(Point3d p1, Point3d p2)
{
    return new Point3d(
        (p1.X + p2.X) / 2,
        (p1.Y + p2.Y) / 2,
        (p1.Z + p2.Z) / 2
    );
}
```

### 오프셋 점 계산

```csharp
public Point3d GetOffsetPoint(Point3d origin, double angle, double distance)
{
    double x = origin.X + distance * Math.Cos(angle);
    double y = origin.Y + distance * Math.Sin(angle);
    return new Point3d(x, y, origin.Z);
}
```

### 다각형 정점 계산

```csharp
public Point3d[] GetPolygonVertices(Point3d center, double radius, int sides)
{
    Point3d[] vertices = new Point3d[sides];
    double angleStep = 2 * Math.PI / sides;

    for (int i = 0; i < sides; i++)
    {
        double angle = i * angleStep;
        vertices[i] = new Point3d(
            center.X + radius * Math.Cos(angle),
            center.Y + radius * Math.Sin(angle),
            center.Z
        );
    }

    return vertices;
}
```

### 세 점으로 호 중심 계산

```csharp
public Point3d? GetArcCenter(Point3d p1, Point3d p2, Point3d p3)
{
    // 간략화된 2D 계산 (Z=0 가정)
    double ax = p1.X, ay = p1.Y;
    double bx = p2.X, by = p2.Y;
    double cx = p3.X, cy = p3.Y;

    double d = 2 * (ax * (by - cy) + bx * (cy - ay) + cx * (ay - by));

    if (Math.Abs(d) < 1e-10)
        return null;  // 세 점이 일직선

    double ux = ((ax * ax + ay * ay) * (by - cy) +
                 (bx * bx + by * by) * (cy - ay) +
                 (cx * cx + cy * cy) * (ay - by)) / d;

    double uy = ((ax * ax + ay * ay) * (cx - bx) +
                 (bx * bx + by * by) * (ax - cx) +
                 (cx * cx + cy * cy) * (bx - ax)) / d;

    return new Point3d(ux, uy, 0);
}
```

### 각도 변환 유틸리티

```csharp
public static class AngleHelper
{
    // 도(Degree) → 라디안(Radian)
    public static double DegreeToRadian(double degree)
    {
        return degree * Math.PI / 180.0;
    }

    // 라디안 → 도
    public static double RadianToDegree(double radian)
    {
        return radian * 180.0 / Math.PI;
    }

    // 각도 정규화 (0 ~ 2π)
    public static double NormalizeAngle(double radian)
    {
        while (radian < 0) radian += 2 * Math.PI;
        while (radian >= 2 * Math.PI) radian -= 2 * Math.PI;
        return radian;
    }
}
```

## 주의사항

- 각도는 항상 **라디안** 단위 사용
- Point3d, Vector3d는 **불변(Immutable)** 타입 - 연산 시 새 객체 반환
- 행렬 곱셈 순서 주의: `A * B`는 B 먼저 적용 후 A 적용

---

*관련: [Entity Manipulation](./entity-manipulation.md) | [Selection](./selection.md)*
