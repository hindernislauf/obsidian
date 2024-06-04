---
cssclasses:
  - dashboard
banner: "![[obsidian_image.png]]"
banner_y: 0.5
---
# 🖥 Route to DE (Hindernislauf)
---

## 🕰 P.A.R.A

- 🚀 **Projects (목표 / 데드라인)**
`$=dv.list(dv.pages('"0. PROJECT"').sort(f=>f.file.ctime,"asc").limit().file.link)`

- 🐢 **Area (책임 / 꾸준히 신경)**
`$=dv.list(dv.pages('"1. AREA"').sort(f=>f.file.ctime,"asc").limit().file.link)`


- 🫠 **Resource (자료 / 관심 분야)**
`$=dv.list(dv.pages('"2. RESOURCE"').sort(f=>f.file.ctime,"asc").limit(25).file.link)`

- 📚 **Archive (완료 / 보관)**
`$=dv.list(dv.pages('"3. ARCHIVE"').sort(f=>f.file.ctime,"asc").limit().file.link)`


---

```dataview
TABLE
	dateformat(file.cday,"yy년 MM월 dd일") as "작성일",
	dateformat(file.mtime,"yy년 MM월 dd일 : hh시 mm분") as "최종 수정 시각"
FROM "/"
WHERE file.folder = "0. PROJECT"
or
file.folder = "1. AREA"
or
file.folder = "2. RESOURCE"
or
file.folder = "3. ARCHIVE"
SORT file.mtime desc
```

