### List git commits between two dates

```bash
git log --since "SEP 1 2020" --until "SEP 30 2020" --pretty=format:"%ai: %s" > project-commits.txt
```

Placeholders that expand to information extracted from the commit:

`%ai` author date, ISO 8601-like format<br>
`%s` subject
