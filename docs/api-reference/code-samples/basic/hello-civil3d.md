# Hello Civil 3D

> 첫 번째 Civil 3D 플러그인 예제

## 프로젝트 설정

1. Visual Studio에서 **클래스 라이브러리(.NET Framework)** 프로젝트 생성
2. 대상 프레임워크: **.NET Framework 4.8**
3. 플랫폼 대상: **x64**

## 참조 추가

```
C:\Program Files\Autodesk\AutoCAD 2025\accoremgd.dll
C:\Program Files\Autodesk\AutoCAD 2025\acdbmgd.dll
C:\Program Files\Autodesk\AutoCAD 2025\acmgd.dll
C:\Program Files\Autodesk\AutoCAD 2025\C3D\AeccDbMgd.dll
```

모든 참조의 **로컬 복사 = False** 설정

## 코드

```csharp
using System;
using Autodesk.AutoCAD.Runtime;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;
using Autodesk.Civil.ApplicationServices;
using Autodesk.Civil.DatabaseServices;

[assembly: CommandClass(typeof(HelloCivil3D.Commands))]

namespace HelloCivil3D
{
    public class Commands
    {
        /// <summary>
        /// 도면의 모든 선형 목록 출력
        /// </summary>
        [CommandMethod("HELLO")]
        public void HelloCommand()
        {
            Document doc = Application.DocumentManager.MdiActiveDocument;
            Editor ed = doc.Editor;
            Database db = doc.Database;

            ed.WriteMessage("\n=== Hello Civil 3D! ===\n");

            try
            {
                CivilDocument civilDoc = CivilApplication.ActiveDocument;

                using (Transaction tr = db.TransactionManager.StartTransaction())
                {
                    // 선형 목록 조회
                    ObjectIdCollection alignmentIds = civilDoc.GetAlignmentIds();
                    ed.WriteMessage($"\n선형 개수: {alignmentIds.Count}");

                    foreach (ObjectId id in alignmentIds)
                    {
                        Alignment alignment = tr.GetObject(id, OpenMode.ForRead) as Alignment;
                        if (alignment != null)
                        {
                            ed.WriteMessage($"\n  - {alignment.Name} (L={alignment.Length:F2}m)");
                        }
                    }

                    // 지형 목록 조회
                    ObjectIdCollection surfaceIds = civilDoc.GetSurfaceIds();
                    ed.WriteMessage($"\n\n지형 개수: {surfaceIds.Count}");

                    foreach (ObjectId id in surfaceIds)
                    {
                        Surface surface = tr.GetObject(id, OpenMode.ForRead) as Surface;
                        if (surface != null)
                        {
                            ed.WriteMessage($"\n  - {surface.Name}");
                        }
                    }

                    tr.Commit();
                }
            }
            catch (System.Exception ex)
            {
                ed.WriteMessage($"\n오류: {ex.Message}");
            }

            ed.WriteMessage("\n\n=== 완료 ===");
        }

        /// <summary>
        /// 선택한 점의 지형 표고 조회
        /// </summary>
        [CommandMethod("GETELEV")]
        public void GetElevationCommand()
        {
            Document doc = Application.DocumentManager.MdiActiveDocument;
            Editor ed = doc.Editor;
            Database db = doc.Database;

            try
            {
                CivilDocument civilDoc = CivilApplication.ActiveDocument;

                // 지형 선택
                ObjectIdCollection surfaceIds = civilDoc.GetSurfaceIds();
                if (surfaceIds.Count == 0)
                {
                    ed.WriteMessage("\n지형이 없습니다.");
                    return;
                }

                using (Transaction tr = db.TransactionManager.StartTransaction())
                {
                    // 첫 번째 TIN 지형 사용
                    TinSurface surface = null;
                    foreach (ObjectId id in surfaceIds)
                    {
                        Surface surf = tr.GetObject(id, OpenMode.ForRead) as Surface;
                        if (surf is TinSurface tin)
                        {
                            surface = tin;
                            break;
                        }
                    }

                    if (surface == null)
                    {
                        ed.WriteMessage("\nTIN 지형이 없습니다.");
                        return;
                    }

                    ed.WriteMessage($"\n사용 지형: {surface.Name}");

                    // 점 선택
                    PromptPointResult ppr = ed.GetPoint("\n점 선택: ");
                    if (ppr.Status != PromptStatus.OK)
                        return;

                    Point3d point = ppr.Value;

                    // 표고 조회
                    try
                    {
                        double elevation = surface.FindElevationAtXY(point.X, point.Y);
                        ed.WriteMessage($"\n좌표: ({point.X:F2}, {point.Y:F2})");
                        ed.WriteMessage($"\n표고: {elevation:F3}m");
                    }
                    catch
                    {
                        ed.WriteMessage("\n선택한 점이 지형 범위를 벗어났습니다.");
                    }

                    tr.Commit();
                }
            }
            catch (System.Exception ex)
            {
                ed.WriteMessage($"\n오류: {ex.Message}");
            }
        }
    }
}
```

## 빌드 및 테스트

1. **빌드**: Release, x64
2. **Civil 3D 실행**
3. **명령행**: `NETLOAD` → DLL 선택
4. **명령 실행**: `HELLO` 또는 `GETELEV`

## 예상 출력

```
=== Hello Civil 3D! ===

선형 개수: 2
  - 중심선 (L=1523.45m)
  - 접속도로 (L=234.12m)

지형 개수: 1
  - 현황지형

=== 완료 ===
```

## 다음 단계

- [플러그인 번들 배포](../../common/plugin-bundle.md)
- [Alignment API](../../civil3d-api/alignment.md)
- [Surface API](../../civil3d-api/surface.md)
