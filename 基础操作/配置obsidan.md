#### bash命令
git --version

git config --global user.name "TODESENGEL1116"
git config --global user.email "t3133724946@gmail.com"

git init
git config --local commit.gpgsign false
git remote add origin https://github.com/TODESENGEL1116/ProgrammingNoteOnObsidian.git
git add .
git commit -m "Initial vault setup"
git push -u origin main