# samuelscheiderich.com
Personal website and public pages.

Deployment is expected to be handled by a gitignored `deploy` executable file/script.

This script should copy/overwrite from the generated `public` folder to the server `public` folder.

One way to do this is with rsync, like `rsync -avz --del public/ <ssh shortcut name, or user@host>:/path-to/public/`.
