# 프로젝트 구조

> Civil 3D 플러그인 솔루션 구성 권장 사항

## 기본 솔루션 구조

```
MyPlugin/
├── MyPlugin.sln
├── src/
│   ├── MyPlugin/                    # 메인 프로젝트
│   │   ├── MyPlugin.csproj
│   │   ├── Commands/                # 명령어 정의
│   │   │   └── MyCommands.cs
│   │   ├── Services/                # 비즈니스 로직
│   │   ├── Models/                  # 데이터 모델
│   │   └── Utils/                   # 유틸리티
│   │
│   └── MyPlugin.UI/                 # WPF UI (선택)
│       ├── MyPlugin.UI.csproj
│       ├── Views/
│       └── ViewModels/
│
├── tests/
│   └── MyPlugin.Tests/
│
└── bundle/                          # 배포용 번들
    └── MyPlugin.bundle/
        ├── PackageContents.xml
        └── Contents/
```

## 명령어 클래스 기본 구조

```csharp
using Autodesk.AutoCAD.Runtime;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;
using Autodesk.Civil.ApplicationServices;
using Autodesk.Civil.DatabaseServices;

// 어셈블리에 명령어 클래스가 있음을 선언
[assembly: CommandClass(typeof(MyPlugin.Commands.MyCommands))]

namespace MyPlugin.Commands
{
    public class MyCommands
    {
        [CommandMethod("MYCOMMAND")]
        public void MyCommand()
        {
            // 현재 문서와 에디터 가져오기
            Document doc = Application.DocumentManager.MdiActiveDocument;
            Editor ed = doc.Editor;
            Database db = doc.Database;

            // Civil 3D 문서
            CivilDocument civilDoc = CivilApplication.ActiveDocument;

            ed.WriteMessage("\nMyCommand 실행됨");
        }
    }
}
```

## 트랜잭션 패턴

```csharp
using (Transaction tr = db.TransactionManager.StartTransaction())
{
    try
    {
        // 작업 수행
        BlockTable bt = tr.GetObject(db.BlockTableId, OpenMode.ForRead) as BlockTable;
        BlockTableRecord btr = tr.GetObject(bt[BlockTableRecord.ModelSpace], OpenMode.ForWrite) as BlockTableRecord;

        // 객체 추가/수정

        tr.Commit();  // 성공 시 커밋
    }
    catch (System.Exception ex)
    {
        ed.WriteMessage($"\n오류: {ex.Message}");
        // 트랜잭션 자동 롤백 (Commit 없이 Dispose)
    }
}
```

## 네임스페이스 참조 요약

### AutoCAD 기본

```csharp
using Autodesk.AutoCAD.Runtime;              // CommandMethod, IExtensionApplication
using Autodesk.AutoCAD.ApplicationServices;  // Application, Document
using Autodesk.AutoCAD.DatabaseServices;     // Database, Transaction, Entity
using Autodesk.AutoCAD.EditorInput;          // Editor, PromptResult
using Autodesk.AutoCAD.Geometry;             // Point3d, Vector3d
```

### Civil 3D

```csharp
using Autodesk.Civil;                        // 공통
using Autodesk.Civil.ApplicationServices;    // CivilApplication, CivilDocument
using Autodesk.Civil.DatabaseServices;       // Alignment, Profile, Surface 등
using Autodesk.Civil.DatabaseServices.Styles; // 스타일
```

## IExtensionApplication 구현 (초기화)

```csharp
public class MyPluginApp : IExtensionApplication
{
    public void Initialize()
    {
        // 플러그인 로드 시 실행
        Editor ed = Application.DocumentManager.MdiActiveDocument?.Editor;
        ed?.WriteMessage("\nMyPlugin 로드됨");
    }

    public void Terminate()
    {
        // 플러그인 언로드 시 실행
    }
}
```

---

*다음: [플러그인 번들 배포](./plugin-bundle.md)*
