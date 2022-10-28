---
layout: single
title:  "python package management with the requirements.txt in IntelliJ"
date:   2022-10-28 09:31:00 +0900
categories: python
header:
    teaser: /docs/assets/2022-10-28-intellij-page.png
---

It's pretty annoying to install packages required for a python project without missing.
Especially when the project is copied to a new server, it could take time to install packages.
The requirements.txt is popularly used for managing packages to remove this inconvenient situation.
IntelliJ provides a python plugin and easy-to-use GUI to handle the requirements.txt.

![IntelliJ python plugin](/docs/assets/2022-10-28-python-plugin.png)

After installing the python plugin, "Sync Python Requirements..." is shown on the Tools menu.
![IntelliJ menu](/docs/assets/2022-10-28-intellij-menu.png)

Just select "Sync Python Requirements,"
and then the "Sync Python Requirements" popup window is shown to indicate the requirements.txt file
and set the package version control policies.

![Sync Python Requirements](/docs/assets/2022-10-28-sync-python-requirements.png)

The following requirements.txt is before applying "Sync Python Requirements."

![Before Requirements.txt](/docs/assets/2022-10-28-before-requirements.png)

Once clicking the OK button on the "Sync Python requirements" popup window,
the requirements.txt file will be changed as below. In addition, not listed packages are shown on the file.
![After Requirements.txt](/docs/assets/2022-10-28-after-requirements.png)

The version control policies using the inequality sign are as below,
[Ref: IntelliJ Official Docuemtns: Manage dependencies using requirements.txt](https://www.jetbrains.com/help/idea/managing-dependencies.html)

| Method | Example |
|---|---|
|Strong equality|Django==3.0.3|
|Greater or equal|Django>=3.0.3|
|Compatible version|Django~=3.0.3|

Python packages on the file can be installed by clicking action buttons on the yellow label on the requirements.txt file.
![Install Packages on the file](/docs/assets/2022-10-28-intellij-page.png)

The following command installs the packages listed on the file.

    pip install -r /path/to/requirements.txt

