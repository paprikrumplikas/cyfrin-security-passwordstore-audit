1. Check name of current branch: 
git branch

2. check out specific commit hash: 
git checkout 7d55682ddc4301a7b13ae9413095feffd9924566

3. switch branch: 
git switch -c passwordstore-audit

4. install code line counter: 
npm install -g cloc 

5. count lines of source code: 
cloc ./src

6. Install dependency Solidity Metrics, and use it to estimate code complexity.
To run: right click on folder on file, and select Solidity Metrics.
To export: start writing ">export" in the command pallet

7. notations in the audited files
// i -- informational
// q -- question
// n -- note
// @audit-high <notes here>

8. check test coverage (note that this is often a vanity metric)
forge coverage

9. write the findings in findings.md

10. write the report.md 

11. generate a pdf from the findings (need Pandoc and LaTex install, and the eisvogel.latex template installed, and pdf viewer as a add-on installed)
https://github.com/Cyfrin/security-and-auditing-full-course-s23/discussions/31

```
cd audit-data
pandoc report.md -o 2024-01-15-password-store-report.pdf --from markdown --template=eisvogel --listings
```
