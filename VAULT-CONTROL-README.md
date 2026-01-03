# VAULT CONTROL README

## Trigger Format
```
notes: {VERB} {OBJECT}
notes: check {OBJECT}
```

## VERBS
- `add` - Add new content
- `update` - Update existing content
- `fix` - Fix issues or errors
- `configure` - Configure settings
- `document` - Document something
- `review` - Review content
- `plan` - Plan something

## RULES

1. **File = {OBJECT}.md**
   - If file exists → update
   - Else → create

2. **Log entries**: one line only

3. **Do not scan vault**

4. **If unsure → ask once**

## NOTE FORMAT

```
{OBJECT}

Action: {VERB} {OBJECT}

Log:
YYYY-MM-DD {entry}
```

## EXAMPLES

- `notes: update pfsense`
- `notes: fix sink`
- `notes: configure vpn`
- `notes: check pfsense`

