# 2. Git & GitHub란?

![Source : [https://opentutorials.org/course/2708/15606](5263.png)

Source : [https://opentutorials.org/course/2708/15606](https://opentutorials.org/course/2708/15606)

### Intro. Version Control의 필요성

버전 관리의 개념을 이해하기 위해서는, 소프트웨어 개발 과정에서 발생할 수 있는 여러 문제들을 상상해보는 것이 좋습니다. 예를 들어, 코드 변경 후 예상치 못한 문제가 발생했을 때, 이전 버전으로 쉽게 돌아갈 수 있다면 얼마나 좋을까요? 버전 관리 시스템은 바로 이러한 필요성에서 출발합니다.

- Version Control System(VCS)은 파일 변화를 시간에 따라 기록하여, 특정 시점에 저장된 상태를 나중에 참조할 수 있도록 하는 시스템입니다.
- 개발 과정에서 코드의 변경 사항을 추적하고, 여러 사람이 작업할 때 충돌을 방지하는 등의 기능을 제공합니다.

예를 들어, 여러 명의 개발자가 함께 협업을 한다고 가정해봅시다. 팀 단위로 코드를 함께 구현하기 위해서는 함께 사용할 수 있는 코드 템플릿이 필요합니다. 그 템플릿을 만들고, 해당 템플릿을 모든 개발자의 각자 컴퓨터에 공유받았습니다.

이제 각 개발자는 본인이 구현해야하는 코드들 작성하고, 테스트를 진행합니다. 이 과정에서 각 개발자들의 코드는 따로따로 수정되었습니다. 이제 이것들을 합쳐서 구현하려고 하는데, 문제가 발생했습니다.

![maxresdefault.jpg](maxresdefault.jpg)

코드를 합치려는데, 서로 수정한 파트가 달라서 하나의 코드로 단순하게 합치는게 어려워진겁니다! 이런 상황들을 수정하기 위해서는 어떻게 해야할까요?

이번에는 서로 대화를 통해 코드를 잘 수정하여 합친 완전한 코드가 만들어졌습니다. 그래서 이 코드를 첫번째 완성본으로 하고, 추가적인 개발을 이어나갔습니다. 한창 개발이 이어져서 두번째, 세번째 완성본도 나오기 시작했습니다. 네번째 완성본을 만들던 도중, 이들은 두, 세번째 추가로 개발한 것들에 문제가 생겨 첫번째 완성본으로 돌아가서 다시 개발을 해야한다는 것을 알게 되었습니다. 이런 경우에는 어떻게 해결해야할까요?

VCS는 두 가지 방법이 있습니다.

- **Central Version Control System(CVCS):**
    - 예: Subversion(SVN), CVS 등. 중앙 서버가 프로젝트의 모든 버전을 관리합니다.
- **`Distributed Version Control System(DVCS)`:**
    - 예: **Git**, Mercurial 등. 각 사용자가 파일의 전체 히스토리를 가진 전체 복제본을 로컬 시스템에 보관합니다.
    - 각자 독립적으로 개발하고, 전체적인 시스템을 관리하는 형태가 더욱 효율적으로 개발이 가능하기 때문에 Git을 주로 사용합니다.

## 1. Git이란?

이제 우리는 버전 관리의 중요성을 이해했습니다. 이제는 가장 널리 사용되는 DVCS인 **Git**에 대해 자세히 알아볼 것입니다. Git은 단순히 코드의 버전을 관리하는 것을 넘어 현대 소프트웨어 개발에서 협업의 핵심 도구입니다.

### **1-1. Git의 기본 개념**

- **Git 소개:**
    - Git은 Linus Torvalds(리눅스 만든 그 아저씨)에 의해 개발된 분산 버전 관리 시스템입니다. 프로젝트의 모든 히스토리와 버전 정보를 저장하여, 네트워크 상태와 무관하게 언제든지 접근할 수 있습니다.
    - 리눅스 커널을 개발할 때 사용하던 기존의 DVCS인 비트키퍼가 무료 사용을 없애자, Linus는 직접 본인이 원하는 기능이 담긴 DVCS를 개발하기 시작했다. 그리고 이것이 Git의 시작이다.
    - Git은 2005년 4월 3일 시작되어, 2024년 4월 현재 2.44 버전까지 출시되었다.
- DVCS로서의 특징 :
    - Git을 사용해서 서버에 있는 같은 코드를 각자 로컬환경에 전체 복사하여 사용이 가능하다.
    - 각자 로컬에서 개발한 코드를 서버에 하나로 합쳐서 사용할 수 있다.
    - 이러한 모든 기록을 versioning하여 관리할 수 있다.

### **1-2. Git의 주요 기능**

![Source : [https://www.youtube.com/watch?app=desktop&v=e9lnsKot_SQ](maxresdefault_(1).jpg)

Source : [https://www.youtube.com/watch?app=desktop&v=e9lnsKot_SQ](https://www.youtube.com/watch?app=desktop&v=e9lnsKot_SQ)

- **주요 명령어와 기능:**
    - `init`: 새로운 Git 저장소를 생성합니다.
    - `clone`: 원격 저장소의 복제본을 로컬에 생성합니다.
    - `add`: 작업 디렉토리의 변경 내용을 스테이징 영역에 추가합니다.
    - `commit`: 스테이징 영역의 변경 내용을 저장소에 기록합니다.
    - `push`: 로컬 저장소의 변경 내용을 원격 저장소에 업로드합니다.
    - `pull`: 원격 저장소의 변경 내용을 로컬 저장소에 가져옵니다.
    - `branch`: 독립적으로 작업을 수행할 수 있는 작업 영역을 생성합니다.
    - `merge`: 다른 브랜치의 변경 내용을 현재 브랜치에 통합합니다.
    - `reset` : commit을 취소하고 이전 상태로 돌아가는 명령어입니다. 이 과정에서 작업 내용을 유지하거나 제거할 수 있습니다.
    - `revert` : commit을 되돌리는 명령어입니다. 새로운 commit을 생성하여 특정 commit으로 변경된 사항을 없던 일로 만듭니다.

**[실습] Visual Studio Code, docker container(Linux), Terminal(CLI)을 사용하여 Git의 기능들을 연습해봅시다.**

![GitHub-Mark-ea2971cee799.png](GitHub-Mark-ea2971cee799.png)

## 2. GitHub란?

Git의 기본을 이해했다면, 이제 이를 기반으로 하는 가장 인기 있는 코드 호스팅 플랫폼인 GitHub에 대해 알아볼 차례입니다. GitHub는 코드를 저장하고 관리할 뿐만 아니라, 전 세계의 개발자들과 협업할 수 있는 강력한 플랫폼입니다.

### **2-1. GitHub의 기본 개념**

- **GitHub 소개:**
    - GitHub은 Git 기반의 온라인 코드 호스팅 서비스로, 코드 버전 관리 및 협업 기능을 제공합니다. 개인 프로젝트부터 대규모 오픈 소스 프로젝트까지 다양한 개발 작업을 지원합니다.
    - Microsoft가 2018년에 75억 달러에 인수하였습니다.
- **GitHub의 역할:**
    - 코드 공유, 프로젝트 관리, 협업을 위한 중심 장소 역할을 하며, 개발자의 포트폴리오로도 활용됩니다.

### **2-2. GitHub의 핵심 기능**

- **코드 호스팅 및 버전 관리:**
    - Git Repository를 원격으로 관리하며, 코드의 버전 관리와 협업을 원활하게 합니다.
    
    ![GitHub Repository를 생성하면 처음 나오는 화면](Screenshot_2024-04-11_at_2.31.50_AM.png)
    
    GitHub Repository를 생성하면 처음 나오는 화면
    
- **Pull Requests (PR):**
    - fork한 repo를 origin repo에 병합하기 위해서 요청을 보내는 기능입니다. 코드 변경 사항을 리뷰하고 논의하는 기능으로, 코드 품질 향상에 기여합니다.
- **Issue Tracking:**
    - 버그 리포트, 기능 요청 등 프로젝트 관련 이슈를 관리할 수 있는 시스템입니다.
- **GitHub Actions:**
    - CI/CD(Continuous Integration/Continuous Deployment)를 위한 자동화된 워크플로우를 구성할 수 있습니다.
- **Wiki and Pages:**
    - 프로젝트 문서를 위한 위키 기능과, GitHub Pages를 통해 정적 웹사이트를 호스팅할 수 있습니다.

### **2-3. GitHub을 통한 협업의 이점**

![Review-Comment-1.webp](Review-Comment-1.webp)

- 투명한 커뮤니케이션:
    - Issue Tracking과 PR을 통해 개발 과정이 투명해지고, 효율적인 커뮤니케이션을 가능하게 합니다.
- Open-source Contribution**:**
    - 전 세계 수많은 오픈 소스 프로젝트에 기여할 수 있는 기회를 제공합니다.
- Peer Code Review**:**
    - PR을 통한 코드 리뷰 과정은 코드 품질 향상과 지식 공유를 촉진합니다.

**몇가지 실제 사례를 살펴봅시다.**

1. [나무위키](https://namu.wiki/w/Git)-Git
2. 전세계에서 가장 인기가 많은 GitHub Repo : ‣
3. [Pytorch](https://github.com/pytorch/pytorch) GitHub Repo
4. https://github.com/freeCodeCamp/freeCodeCamp

**[실습] Visual Studio Code, docker container(Linux), Terminal(CLI), Git을 사용하여 GitHub의 기능들을 연습해봅시다.**

GitHub Repo : [https://github.com/idaeschool/yeardream_04](https://github.com/idaeschool/yeardream_04)