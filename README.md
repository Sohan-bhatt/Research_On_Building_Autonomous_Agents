# Research On Building Autonomous Agents

This project publishes documentation through **GitHub Wiki**.

## Wiki Repositories and URLs

- Main repo: `https://github.com/Sohan-bhatt/Research_On_Building_Autonomous_Agents.git`
- Wiki git repo: `https://github.com/Sohan-bhatt/Research_On_Building_Autonomous_Agents.wiki.git`
- Live wiki page: `https://github.com/Sohan-bhatt/Research_On_Building_Autonomous_Agents/wiki`

## Important

- Wiki content is stored in the separate `.wiki.git` repo.
- Pushing to `main`/`master` in the main repo does not update wiki pages.
- To let teammates push wiki pages, add them as collaborators with write access.

## Standard Wiki Workflow

1. Clone wiki repo:
   ```bash
   git clone https://github.com/Sohan-bhatt/Research_On_Building_Autonomous_Agents.wiki.git
   cd Research_On_Building_Autonomous_Agents.wiki
   ```
2. Edit or add markdown pages (`.md`), including `Home.md` and `_Sidebar.md`.
  ### sync
   ```bash
      # Sync all notes markdown files (keeps directory structure)
    rsync -av --include="*/" --include="*.md" --exclude="*" \
      ../my-notes/notes/ ./

    # Sync wiki helper pages (Home, _Sidebar, etc.)
    rsync -av --include="*.md" --exclude="*" \
      ../my-notes/wiki/ ./

   ```
3. Commit and publish:
   ```bash
   git add .
   git commit -m "Update wiki pages"
   git push origin master
   ```


## Local Source Notes

- `notes/` contains local source notes used to draft content.
- `wiki/` can hold reusable wiki helper templates/pages.
