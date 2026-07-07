# GitHub Pages

이 저장소는 MkDocs Material과 GitHub Pages로 문서를 게시할 수 있습니다.

## 로컬 미리보기

문서 의존성을 설치합니다.

```bash
pip install -r docs/requirements.txt
```

로컬 문서 서버를 실행합니다.

```bash
mkdocs serve -f docs/mkdocs.yml
```

정적 사이트를 빌드합니다.

```bash
mkdocs build --strict -f docs/mkdocs.yml
```

생성된 사이트는 `site/` 디렉터리에 저장됩니다.

## 게시 흐름

```text
docs/en/*.md
docs/ko/*.md
docs/mkdocs.yml
docs/requirements.txt
        |
        v
GitHub Actions
        |
        v
GitHub Pages
```
