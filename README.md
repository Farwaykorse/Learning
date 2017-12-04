<!-------------------------------------------------------------><a id="top"></a>
# Notes on Programming (...), notes
<!----------------------------------------------------------------------------->

<!-- introduction -->
These pages contain documentation, notes, ideas, strategies and examples,
created while studying topics of personal interest.  
All written in English <!-- for now --> with consistant formatting in markdown
to be usable both from the web and in plain-text.

The main focus is (for now) on programming and related content, but this can
change and in potential anything could be added to this.

##### TOC #####
- [Content](#content)  
- [File Organization and Formatting](#organize)  
- [Commit Strategy](#commits)  
<!-- - [Contributing](#contributing)  -->
<!-- - [License](#license)  -->

<!---------------------------------------------------------><a id="content"></a>
## Content
<!----------------------------------------------------------------------------->

- [C++](./cpp/README.md)  
- [git](./git/README.md)  


<!------------------------------------------------------><a id="organize"></a>
## File Organization and Formatting
<!----------------------------------------------------------------------------->
### Directory Structure
Grouping content by application or programming language in directories.
Each containing a `README.md` functioning as an entry point
(on GitHub these are displayed upon opening a directory).

### Files
Only a single subject per file.
This enforces clear definitions and scope.
Little initial content is not a problem, it invites adding more detail.

### Filenames and Titles
Use short expressive filenames covering the content. A more exhaustive name
can be used in the page title. If the title isn't covering the whole of the
content, move it to its own page and link to it.

<!------------------------------------------------------><a id="formatting"></a>
### Formatting
- Keep lines within 80 characters. Except:
    - weblinks
    - quotes
    - other; prefer placing these between ` `` `
- Start with a header, introduction and table of contents (TOC).
- End with a horizontal line and a link to the top of the document.
- Mark headers with continues comment `<!------>` bars, over the full 80 chars.
- Headers:
    - Level 1 `#` only for the document title
    - Level 2 `##` for headers in the document  
      comment bars above and below.
    - Level 3 `###` and more for subtiltes and internal headers.  
      When important, only a comment bar with anchor id above.
    - Give each header a locally unique anchor id (in the bar)
    - Add each header to the TOC.
- TODO's: hide these in comments `<!-- -->`.
<!-- TODO:
- Linking other files?
- Linking to relevant documents?? where?
- External links | references
    - when inline?
    - inline formatting?
    - seperate links section? Wikipedia style?
    - differentiate between direct links (page) and general links (website)
- Remarks?? Notes??
- Todo | planning to write about
- ...
-->

#### Example:
````
<!-------------------------------------------------------------><a id="top"></a>
# Title
<!----------------------------------------------------------------------------->

<!-- introduction -->

##### TOC #####
- [Header_name](#header_id)  
  - [Sub_name](#sub_id)  

<!-------------------------------------------------------><a id="header_id"></a>
## Header_name
<!----------------------------------------------------------------------------->
<!-- content -->

<!----------------------------------------------------------><a id="sub_id"></a>
### Sub_name
<!-- more content -->

-----------
[top](#top)
````

<!--------------------------------------------------------><a id="commits"></a>
## Commit strategy
<!----------------------------------------------------------------------------->
**Master branch**  
Clean versions go into the master branch and are displayed online.

**Other branches**  
Changes go into working branches and are commited and pushed often, as remote
backups.

<!----------------------------------------------------><a id="contributing"></a>
<!-- Contributing -->
<!----------------------------------------------------------------------------->

<!--------------------------------------------------------><a id="license"></a>
<!-- License -->
<!----------------------------------------------------------------------------->

-----------
[top](#top)
