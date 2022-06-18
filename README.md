# easy2edm

:warning:This is in progress and not fully functional:warning:

This repo was forked from [easy2acl](https://github.com/acl-org/easy2acl) to better meet the needs of the [International Conference on Educational Data Mining](https://educationaldatamining.org/conferences/).

Major differences:

- One proceedings for multiple tracks
- Different front matter
- ACL Anthology support largely removed


## Prerequisites

The Python 3 `unicode_tex` package is needed and can be installed using pip (`pip3 install -r requirements.txt`).
The tar command is also needed (and should be available at PATH).

## Prepare data

Each track **that exists in EasyChair** has its own subfolder:

- Full papers
- Short papers
- Posters
- Doctoral consortium
- Industry track

The subfolder file structure should look like this:

    |-- meta                # conference metadata
    |-- submissions         # copied list of submissions
    |-- submission.csv      # (optional) to include abstracts
    |-- accepted            # copied list of accepted papers
    `-- pdf
        |-- ${abbrev}_${year}_paper_1.pdf
        |-- ${abbrev}_${year}_paper_2.pdf
        `-- ...

(where `${abbrev}` and `${year}` are defined in the `meta` file, see below).

Perform the following steps to create each subfolder.
**You must have `track chair` level access to perform all of these steps.**

1. Download submissions

Start by downloading the actual submissions: In EasyChair, go to the page _Submissions_ and click the link _Download submissions_ found in the upper right hand side. Extract the PDF files to a folder `pdf`. Rename the files appropriately, e.g. using Thunar's batch renaming tool.

2. Write the track metadata file

The `meta` file defines a number of conference-level values that are needed to generate the BibTeX and to interpret the file names.
Information on the layout and format of this file can be found on [this page in the ACLPUB documentation](https://acl-org.github.io/ACLPUB/anthology.html#the-meta-file).

An example file can be found [here](example-files/meta).
However, you are best off using EDM examples, because they differ slightly from ACL.

3. Get submissions metadata

On the same page _Submissions_: In the table, starting with the first submission entry (excluding the first row/header starting with `#`), select and copy the entire table. Copy and paste this into a proper text editor of your choice and save the file as `submissions`. Remember to not force any linebreaks. Each row in the table should correspond to one line in the resulting file. A sample `submissions` file is available [here](example-files/submissions).

We now have information about all the submissions but not whether they are accepted or not. Of course we do not want to include the rejected submissions. We need to get one more piece of information.

4. Get acceptance metadata

Go to _Status -> All submissions_.
Here we find the information on what submissions are accepted.
Copy the content of this table as you did with the previous one. Save the content as `accepted`, and make sure that each row in the table corresponds to one line in the resulting file.
Accept codes must match ACL codes, e.g. ACCEPT.
A sample `accepted` file is available [here](example-files/accepted).
**Please note**: the generated proceedings will order the papers in the order they are found in this file.
Please reorder the entires in `accepted' according to the order you would like them displayed (typically, the order they are presented in the program).


5. Download abstracts (requires EasyChair Premium)

Go to _Premium -> Track data download -> CSV data, or download only a subset of these tables click here -> submissions.csv_.
Place this file in the directory as described above.
If present, the abstracts will be generated into the BibTeX files for ingestion by the Anthology scripts.

## First run of script

Before you run the script for the first time, create a dummy pdf file for the full volume consolidated PDF file and the front matter proceedings file using the above file naming convention, and put these in the top `pdf` subfolder.
We will replace the dummy files later and repeat the procedure.

Now run `easy2edm.ipynb`. It will populate the `proceedings` first with accepted pdfs and their bibtex.

    |-- proceedings/
        `-- cdrom/
            |-- ${abbrev}-${year}.bib     # all bib entries
            |-- ${abbrev}-${year}.pdf     # entire volume
            |-- bib/
            |   |-- {year}.{abbrev}-{volume}.0.bib  # frontmatter
            |   |-- {year}.{abbrev}-{volume}.1.bib  # first paper
            |   |-- {year}.{abbrev}-{volume}.2.bib  # second paper
            |   `-- ...
            `-- pdf/
                |-- {year}.{abbrev}-{volume}.0.pdf  # frontmatter
                |-- {year}.{abbrev}-{volume}.1.pdf  # first paper
                |-- {year}.{abbrev}-{volume}.2.pdf  # second paper
                `-- ...

## Combined proceedings

It also makes the file `book-proceedings/all_papers.tex` which contains an index of files. This is read in automatically by `book-proceedings.tex` to generate a full volume consolidated PDF file.

You must edit the front matter in `book-proceedings.tex` and then recompile it. During compiling, it automatically creates a table of contents and includes all papers from `all_papers.tex`.

Once it is compiled, you need to copy `book-proceedings.pdf` and extract the frontmatter section to create is bibtex by rerunning the above script.

- Copy `book-proceedings.pdf` to `${abbrev}_${year}.pdf` in your top `pdf` folder.
- Extract the front matter pages with roman page numbers from `book-proceedings.pdf` and save as `${abbrev}_${year}_frontmatter.pdf` in your top `pdf` folder.
- Re run the script for generating bibtex

## Citation stamping

To add citation info to the left corner of each page, 

- Generate doi for each submission and insert that into the bibtex
- Update the ...
- Run the script

## Additional information

It is your responsibility to make sure that the input files above are created correct.
Easychair may change, and the author(s) of this script make no claims that this script works as intended.
Below are some things to look out for regarding the data you get from EasyChair, and the assumptions made by the script.

* **Title of submission in EasyChair does not match title in the submitted PDF.** In case of a substantial change to the title, and depending on the policy of your conference, you might want to contact the Program Chair.  You might want to do so anyways in case the title is used anywhere else, for example in the conference program.

* **Order of authors of submission in EasyChair does not match the order in the submitted PDF.**

* **Author has multiple names before the last name, e.g. `<first> <middle> <last>`.** This can cause problems with the order of the papers since they are written in alphabetical order according to the first author's last name. The script assumes the format `<first> [<first> <first> ...] <last>`.

* **Some diacritics and special characters in names are not converted by the script.** Certain characters that you expected to be translated into LaTeX escape codes, but were not, might be because they are not handled in the unicode_tex package.
  Make sure that the name was properly written in EasyChair; it might be that the person who entered the name forgot to add diacritics.
  If you want to be nice, you can check the names in your resulting `bib/` files against the names of the actual submissions and make the appropriate changes.

