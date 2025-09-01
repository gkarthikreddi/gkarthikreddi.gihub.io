---
layout: post
title: LibreOffice GSoC'25 Final Report
date: 2025-08-31 06:21:36 +0530
permalink: /libreoffice/gsoc-final-report
---

Hello everyone 👋

I had a great experience participating in this year’s Google Summer of Code (GSoC) with LibreOffice.  
It's a great experience to collaborate with people from all over the world, contribute to an open-source project, and learned a lot along the way.

## Overview
This [project](https://wiki.documentfoundation.org/Development/GSoC/Ideas#New_dialog_to_edit_Table_Styles) is about creating a new dialog to edit **Table Styles** (actually autoformats, which are predefined templates). The dialog was based on the UX team's [proposal](https://design.blog.documentfoundation.org/2015/12/13/style-your-tables/).

**Current Limitations**:
-  In both Writer and Calc, users could not edit an existing table style. The only option was to [format](https://books.libreoffice.org/en/WG76/WG7613-Tables.html#toc27) a table manually and then save it as a new style, but that's not very intuitive.
- Even though the underlying data structure for autoformats is similar in Writer and Calc, each has its own implementation, including a separate "AutoFormat Dialog".
- Autoformats are stored in a binary file which was hard to maintain and not backwards compatible.
- Applying autoformat did not follow the ODF(Open Document Format) specification.

### Project Outcomes
**AutoFormat Dialog**: Now the dialog is shared between Writer and Calc. Nothing is changed visually except the addition of *New Style* and *Edit Style* buttons.

![AutoFormat Dialog](/assets/autoformat_dlg.png){: width="550" }{:style="display:block; margin-left:auto; margin-right:auto;"}

**Table Style Dialog**:
- Added inheritance feature to table styles, every style by default inherits from *Default Style*.
- Users can customize styles for 16 elements (first-row, last-row, body, etc). The styles are applied according to ODF priority.

![Table Styles Dialog](/assets/table_styles_dlg.png){: width="650" }{:style="display:block; margin-left:auto; margin-right:auto;"}

**Adapting ODF specification**:
Table Styles are now applied according to element priority.
For example:
- The first row of a table is formatted by the style in *first-row* element.
- If no background color is defined, it falls back to the *odd-row* style.
- If that is also undefined, it falls back to the *body* style, and so on.

**Styles Sidebar**: We can edit/create table styles from the Styles Sidebar in Writer.

![Writer Styles Sidebar](/assets/sidebar.png){: width="750" }{:style="display:block; margin-left:auto; margin-right:auto;"}

## Technical Details
Although the project is about creating a dialog, the hardest part was unifying the autoformat code from Writer and Calc. Writer and Calc have `SwTableAutoFormatTable` and `ScAutoFormat` classes, respectively for handling autoformats, which are required for table styling. I created a new `SvxAutoFormat` class which combines the common functionality of Writer and Calc. Now this new class is inherited by both Writer and Calc classes.

![SvxAutoFormat](/assets/svxautoformat.png){: width="650" }{:style="display:block; margin-left:auto; margin-right:auto;"}

**Loading Table Styles**: Becasue we wanted to replace the old binary file for loading/storing table styles, I created an XML file ('tablestyles.xml') based on existing table styles which are shipped with LibreOffice. I created `SvxTableStylesImport` class which is used to load styles to `SvxAutoFormat` and `SvxTableStylesExport` class to save the `SvxAutoFormat` to the XML file.

**Other Improvements**:
- New tables are created with *Default Style* by default. There is no longer a style called 'None' in the 'AutoFormat Dialog'.
- Bugs related to autoformat have been fixed
  - [Writer:Insert Table with style still directly applies Liberation Serif 12](https://bugs.documentfoundation.org/show_bug.cgi?id=121023)
  - [Inserting or removing a row/column changes entire table's formatting](https://bugs.documentfoundation.org/show_bug.cgi?id=126008)
  - [if you add rows/columns to a 1x1 table, the cell borders are not displayed](https://bugs.documentfoundation.org/show_bug.cgi?id=166223)(Because by default every new table has *Default Style*)

**Related Patches**:
- [SvxAutoFormat and dialogs](https://gerrit.libreoffice.org/c/core/+/190024/)
- [Adapting Writer](https://gerrit.libreoffice.org/c/core/+/190024/10)
- [Adapting Calc](https://gerrit.libreoffice.org/c/core/+/190026)

### What's Left
- Add WYSIWYG preview of table styles in the Writer styles sidebar.
- Add UI tests to verify the workflow.

### Limitations
- **Backward compatibility**: Old `.odt` files open correctly, but since the internal structure of table styles has changed to follow ODF, when you interact with the table (inserting/deleting rows/columns), the style of the table changes.
- **Number formatting** in the Table Styles dialog is not yet functional (planned fix).

### Future Scope
- In Impress, the Table styles panel has checkboxes to enable/disable certain element styles, I have already implemented the backend functionality, depending on the UX team's approval, this can be added to the new dialog or sidebar.

![Impress Table Styles](/assets/impress_table_styles.png){: width="400" }{:style="display:block; margin-left:auto; margin-right:auto;"}

--

I'm very grateful to the entire community. Special thanks to my mentors, Rafael Lima and Heiko Tietze, for their patience and support throughout the process. Looking forward to contributing more to the community!🚀


