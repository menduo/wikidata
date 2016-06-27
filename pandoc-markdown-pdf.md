---
categories:pandoc, markdown, pdf
...

## Now

```
pandoc 1.15.0.6
Compiled with texmath 0.8.2.2, highlighting-kate 0.6.
MacTeX-2015
```

## Inlstall

```
# http://tug.org/mactex/mactex-download.html or 
brew cask install mactex
```

## cleanup

```bash
brew cask cleanup
rm -rf /opt/homebrew-cask/Caskroom/mactex/
```

```
\newcommand{\tightlist}{%
  \setlength{\itemsep}{0pt}\setlength{\parskip}{0pt}}
```


```bash
function md2pdf(){
    _pan_temp_pdf=~/Dropbox/softs/pandocs/temp-pdf
    _pan_temp_listing=~/Dropbox/softs/pandocs/temp-listings.tex
    pandoc --latex-engine=xelatex \
        $1 -o $1.pdf \
        -f markdown \
        --template=$_pan_temp_pdf \
        --listings -H $_pan_temp_listing -V toc-depth=5 --toc
}
```