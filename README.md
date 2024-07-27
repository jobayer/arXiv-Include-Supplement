# arXiv-Include-Supplement

A method to add external supplementary files with cross-referencing enabled with the `xr` package

Including external supplementary files in arXiv can be a complicated process, especially when using the `xr` package for cross-referencing. In this repository, I am showing a simple way to achieve this. Part of the method has been taken from the [IncludeSupplement](https://github.com/agdelma/IncludeSupplement) repository with slight modification.

# Method

I will divide the whole process into two parts:

1. Enabling cross-referencing from the supplementary file
2. Adding the supplementary file at the end of the main document

For convenience, I will assume the main tex file is named `main.tex` and one supplementary file is named `supp.tex`. You can add more supplementary files as needed.

> Before proceeding, download a copy of your source code as a zip file from Overleaf [**as shown here**](https://www.overleaf.com/learn/how-to/How_do_I_download_the_automatically_generated_files_(e.g._.bbl%2C_.aux%2C_.ind%2C_.gls)_for_my_project%3F_My_publisher_asked_me_to_include_them_in_my_submission) (this is important!) and extract it to your local computer. Delete the `supp.tex` file, as we do not need it. We will need to add several files in this directory and the final contents of this directory will be uploaded on arXiv. For convenience, we will assume that the name of the zip file of the code is `source.zip` and you have downloaded it into the `C:\Users\YourName\Downloads\` directory of your Windows machine.

> Unless specified, all the changes in the code will be made on the local version you just downloaded, not on the code available on Overleaf.

## Enabling cross-referencing

Enabling cross-referencing using the [xr package](https://www.ctan.org/pkg/xr) is one of the most common ways. As I used Overleaf as my LaTeX editor, I followed the instructions as mentioned [here](https://tex.stackexchange.com/a/551218) and I did not test it in my local environment. However, this should work fine in the local environment as well. To enable cross-referencing, you can follow the steps as follows:

1. Add the `xr` package in the preamble of `main.tex` file.

    ```latex
    \usepackage{xr}
    ```

2. Add these helper functions in the preamble of `main.tex` file.

    ```latex
    \makeatletter
    \newcommand*{\addFileDependency}[1]{
    \typeout{(#1)}
    \@addtofilelist{#1}
    \IfFileExists{#1}{}{\typeout{No file #1.}}
    }
    \makeatother

    \newcommand*{\myexternaldocument}[1]{%
    \externaldocument{#1}%
    \addFileDependency{#1.tex}%
    \addFileDependency{#1.aux}%
    }
    ```

3. Add your supplementary files (the `.tex` files) in the preamble of `main.tex` file.

    ```latex
    \myexternaldocument{supp} % note that the .tex extention is not included
    ```

4. Go to Overleaf, navigate to the `supp.tex` file, and compile it. Then download the `output.aux` file from the automatically generated files as shown [here](https://www.overleaf.com/learn/how-to/View_generated_files), rename the `output.aux` file to `supp.aux`, and place it into the specified path as mentioned in step 3 of the local source code you downloaded in your local machine.

    For example, suppose you have the supplementary file `supp.tex` in the same directory as `main.tex` and use the same code shown in step 3. In that case, you will place the `supp.aux` file into the `C:\Users\YourName\Downloads\source` directory.

## Adding The Supplementary File

There are several ways to add external tex files as PDF in the main file. However, arXiv uses `pdfpages` package to add the external documents and we will follow the same procedure.

1. Add the following packages in the preamble of the `main.tex` file.

    ```latex
    \usepackage{pdfpages} % include pdfs
    \usepackage{pgffor} % for loops
    ```

2. From the Overleaf editor, go to the `supp.tex` file and compile it, download it, rename it to `supp.pdf` (or any other name of your like) and then move it to the same directory `C:\Users\YourName\Downloads\source`.

3. Add the following codes in the preamble of the `main.tex` file.

    ```latex
    % Fix for a pdfpages rotation bug with revtex
    \makeatletter
    \AtBeginDocument{\let\LS@rot\@undefined}
    \makeatother

    % the name of the supplement PDF file
    \def\supplementfilename{supp.pdf}

    % Determine the number of pages 
    % in the supplement file and store
    \pdfximage{\supplementfilename}
    \def\numbersupplementpages{\the\pdflastximagepages}

    \newif\ifarXiv
    \arXivtrue
    ```

    > Please note that here, the `supp.pdf` is the name of the supplementary file you just downloaded from Overleaf and moved into the local source directory. So, do not forget to update the file name in the code if you have named it as something else.

4. Add the following code just before the `\end{document}` line of your code.

    ```latex
    \clearpage

    \ifarXiv
        \foreach \x in {1,...,\numbersupplementpages}
        {
            \includepdf[pages={\x}, fitpaper=true]{\supplementfilename}
        }
    \fi
    ```

    You can comment out the `\clearpage` line if you do not want to start your supplementary after the end of the content of your main file.

# Issues Related to SVG Files

If you have SVG files in your project, you will most likely encounter an error while arXiv processes your files. Unlike Overleaf, arXiv does not process the SVG files automatically. You will also need to provide the `*-tex.pdf` and `*-tex.pdf_tex` files of the SVG files to arXiv. I am curious to know if there are any better ways to do this; however, below, I have provided a simple way to resolve the error. But this process might be tiresome if you have a lot of SVG files.

- If you run your LaTeX project on the local machine, then the SVG package should automatically create a directory called `svg-inkscape`, containing all the required `*-tex.pdf` and `*-tex.pdf_tex` files. All you need to do is copy the `svg-inkscape` directory along with all of its contents into the `C:\Users\YourName\Downloads\source` directory.

- However, if your project is on Overleaf, then go to each of the `tex` files available on your project, compile it, and then download all the `*-tex.pdf` and `*-tex.pdf_tex` files from the generated files as shown [here](https://www.overleaf.com/learn/how-to/View_generated_files). Then, inside the `C:\Users\YourName\Downloads\source` directory, create another directory called `svg-inkscape` and paste all the downloaded files. Then, your project should be ready to be processed by arXiv.

# Submitting on arXiv

If you have followed this far, in the directory of your code in your local machine, along with other files, you should have the modified `main.tex`, the `main.bbl`, `supp.aux` and `supp.pdf` files available. Make a zip file containing all the contents available in the `C:\Users\YourName\Downloads\source` directory and upload the zip file and then proceed next to compile your code!

# Bonus Tips! (Optional)

When we refer to something of the supplementary file in our main file, it is more convenient, especially for the scientific articles, to let the reader know that the referenced figure is a part of the supplementary file. One common way to do this is to add **S** before the number of the referenced figure. For example, if I want to cite the _second figure_ of the supplementary file in the main file, writing like `Figure S2` would be convenient. Additionally, one may want to change the `Figure` text shown before the figure number. For example, to indicate a supplementary figure, one may want to write `Supp. Figure 1` instead of `Figure 1` to distinguish from the main figure 1.

1. To add custom text (`S` in this case) before the figure number, you need to add the following in the preamble of your supplementary file, `supp.tex`.

    ```latex
    \renewcommand{\thefigure}{S\arabic{figure}}
    ```

2. To add custom text (`Supp. Figure` in this case) instead of standard `Fig.` or `Figure`, you can try adding the following line in the preamble of your supplementary file, `supp.tex`.

    ```latex
    \renewcommand{\figurename}{Supp. Figure}
    ```

That's it! Thank you for going through this long reading. I tried to describe it as elaborately as possible. _For any queries, feel free to reach me at [jobayer@ieee.org](mailto:jobayer@ieee.org)._
