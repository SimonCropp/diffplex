DiffPlex ![Build](https://github.com/mmanela/diffplex/actions/workflows/dotnet.yml/badge.svg) [![DiffPlex NuGet version](https://img.shields.io/nuget/v/DiffPlex.svg)](https://www.nuget.org/packages/DiffPlex/)
========

DiffPlex is C# library to generate textual diffs. It targets `netstandard1.0+`.

# About the API

The DiffPlex library currently exposes several interfaces and classes for generating diffs:

* `IDiffer` (implemented by the `Differ` class) - This is the core diffing class.  It exposes the low level functions to generate differences between texts.
* `ISidebySideDiffer` (implemented by the `SideBySideDiffer` class) - This is a higher level interface.  It consumes the `IDiffer` interface and generates a `SideBySideDiffModel`.  This is a model which is suited for displaying the differences of two pieces of text in a side by side view.
* `UnidiffRenderer` - A renderer class that generates unified diff (unidiff) format output compatible with Git, patch utilities, and other standard diff tools.

## Examples

For examples of how to use the API please see the following projects contained in the DiffPlex solution.

For use of the `IDiffer` interface see:

* `SidebySideDiffer.cs` contained in the `DiffPlex` Project.
* `UnidiffFormater.cs` contained in the `DiffPlex.ConsoleRunner` project.

For use of the `ISidebySideDiffer` interface see:

* `DiffController.cs` and associated MVC views in the `WebDiffer` project
* `TextBoxDiffRenderer.cs` in the `SilverlightDiffer` project

For use of the `UnidiffRenderer` class see:

* `Program.cs` in the `DiffPlex.ConsoleRunner` project

## Sample code

```csharp
var diff = InlineDiffBuilder.Diff(before, after);

var savedColor = Console.ForegroundColor;
foreach (var line in diff.Lines)
{
    switch (line.Type)
    {
        case ChangeType.Inserted:
            Console.ForegroundColor = ConsoleColor.Green;
            Console.Write("+ ");
            break;
        case ChangeType.Deleted:
            Console.ForegroundColor = ConsoleColor.Red;
            Console.Write("- ");
            break;
        default:
            Console.ForegroundColor = ConsoleColor.Gray; // compromise for dark or light background
            Console.Write("  ");
            break;
    }

    Console.WriteLine(line.Text);
}
Console.ForegroundColor = savedColor;
```

## IDiffer Interface 

```csharp 
/// <summary>
/// Provides methods for generate differences between texts
/// </summary>
public interface IDiffer
{
    /// <summary>
    /// Create a diff by comparing text line by line
    /// </summary>
    /// <param name="oldText">The old text.</param>
    /// <param name="newText">The new text.</param>
    /// <param name="ignoreWhiteSpace">if set to <c>true</c> will ignore white space when determining if lines are the same.</param>
    /// <returns>A DiffResult object which details the differences</returns>
    DiffResult CreateLineDiffs(string oldText, string newText, bool ignoreWhiteSpace);

    /// <summary>
    /// Create a diff by comparing text character by character
    /// </summary>
    /// <param name="oldText">The old text.</param>
    /// <param name="newText">The new text.</param>
    /// <param name="ignoreWhitespace">if set to <c>true</c> will treat all whitespace characters are empty strings.</param>
    /// <returns>A DiffResult object which details the differences</returns>
    DiffResult CreateCharacterDiffs(string oldText, string newText, bool ignoreWhitespace);

    /// <summary>
    /// Create a diff by comparing text word by word
    /// </summary>
    /// <param name="oldText">The old text.</param>
    /// <param name="newText">The new text.</param>
    /// <param name="ignoreWhitespace">if set to <c>true</c> will ignore white space when determining if words are the same.</param>
    /// <param name="separators">The list of characters which define word separators.</param>
    /// <returns>A DiffResult object which details the differences</returns>
    DiffResult CreateWordDiffs(string oldText, string newText, bool ignoreWhitespace, char[] separators);

    /// <summary>
    /// Create a diff by comparing text in chunks determined by the supplied chunker function.
    /// </summary>
    /// <param name="oldText">The old text.</param>
    /// <param name="newText">The new text.</param>
    /// <param name="ignoreWhiteSpace">if set to <c>true</c> will ignore white space when determining if chunks are the same.</param>
    /// <param name="chunker">A function that will break the text into chunks.</param>
    /// <returns>A DiffResult object which details the differences</returns>
    DiffResult CreateCustomDiffs(string oldText, string newText, bool ignoreWhiteSpace, Func<string, string[]> chunker);

            /// <summary>
        /// Create a diff by comparing text line by line
        /// </summary>
        /// <param name="oldText">The old text.</param>
        /// <param name="newText">The new text.</param>
        /// <param name="ignoreWhiteSpace">if set to <c>true</c> will ignore white space when determining if lines are the same.</param>
        /// <param name="ignoreCase">Determine if the text comparision is case sensitive or not</param>
        /// <param name="chunker">Component responsible for tokenizing the compared texts</param>
        /// <returns>A DiffResult object which details the differences</returns>
        DiffResult CreateDiffs(string oldText, string newText, bool ignoreWhiteSpace, bool ignoreCase, IChunker chunker);
}
```

## IChunker Interface

```csharp
public interface IChunker
{
    /// <summary>
    /// Dive text into sub-parts
    /// </summary>
    string[] Chunk(string text);
}
```

Currently provided implementations:

- `CharacterChunker`
- `CustomFunctionChunker`
- `DelimiterChunker`
- `LineChunker`
- `LineEndingsPreservingChunker`
- `WordChunker`

## UnidiffRenderer Class

The `UnidiffRenderer` class provides functionality to generate unified diff (unidiff) format output, which is the standard format used by Git, patch utilities, and other diff tools.

```csharp
// Static method for simple usage
string unidiff = UnidiffRenderer.GenerateUnidiff(
    oldText: "old content", 
    newText: "new content",
    oldFileName: "file1.txt",
    newFileName: "file2.txt"
);

// Instance usage with custom settings
var renderer = new UnidiffRenderer(contextLines: 5);
string unidiff = renderer.Generate(oldText, newText, "before.txt", "after.txt");
```

Key features:
- Generates standard unified diff format compatible with Git and patch tools
- Configurable number of context lines around changes
- Support for custom file names in diff headers  
- Options to ignore whitespace and case differences

Example output:
```
--- before.txt
+++ after.txt
@@ -1,4 +1,4 @@
 Line 1
-Old line 2
+New line 2  
 Line 3
 Line 4
```

## ISideBySideDifferBuilder Interface

```csharp
/// <summary>
/// Provides methods that generate differences between texts for displaying in a side by side view.
/// </summary>
public interface ISideBySideDiffBuilder
{
    /// <summary>
    /// Builds a diff model for  displaying diffs in a side by side view
    /// </summary>
    /// <param name="oldText">The old text.</param>
    /// <param name="newText">The new text.</param>
    /// <returns>The side by side diff model</returns>
    SideBySideDiffModel BuildDiffModel(string oldText, string newText);
}
```

## Sample Website

DiffPlex also contains a sample website that shows how to create a basic side by side diff in an ASP MVC website.

![Web page sample](./images/website.png)

## Sample Blazor App

DiffPlex includes a Blazor sample application demonstrating how to render textual diffs in modern Blazor applications. The sample showcases both server-side and client-side rendering capabilities with interactive diff visualization.

![image](https://github.com/user-attachments/assets/862030f5-c759-4556-8b10-19cc281c5e7d)


The Blazor components provide:
- Interactive side-by-side diff rendering
- Inline diff view mode
- Unififf rendering 

![image](https://github.com/user-attachments/assets/d91f26d6-da4a-47fd-b490-daeb65ddad11)

To run the sample Blazor application:
```bash
cd DiffPlex.Blazor
dotnet run
```

# Windows app

There are 2 libraries for Windows app development.
One is for Windows App SDK, another is for WPF and WinForms.

## WinUI 3 Elements

[![NuGet](https://img.shields.io/nuget/v/DiffPlex.Windows.svg)](https://www.nuget.org/packages/DiffPlex.Windows/)

DiffPlex WinUI library `DiffPlex.Windows` is used to render textual diffs in your app which targets to Windows App SDK.

```csharp
using DiffPlex.UI;
```

And insert following code into the root node of your xaml file, e.g. user control, page or window.

```
xmlns:diffplex="using:DiffPlex.UI"
```

- `DiffTextView` Textual diffs view element.

For example.

```xaml
<diffplex:DiffTextView x:Name="DiffView" />
```

```csharp
DiffView.SetText(OldText, NewText);
```

![WinUI sample](./images/wasdk_split_dark.jpg)

You can also customize the style.
Following are some of the properties you can get or set.

```csharp
// true if it is in split view; otherwise, false, in unified view.
public bool IsSplitView { get; set; }

// true if it is in unified view; otherwise, false, in split view.
public bool IsUnifiedView { get; set; }

// The selection mode of list view. Default is None.
public ListViewSelectionMode SelectionMode { get; set; }

// true if ignore white spaces; otherwise, false. Default is true.
public bool IgnoreWhiteSpace { get; set; }

// true if the text is case sensitive; otherwise, false. Default is false.
public bool IsCaseSensitive { get; set; }

// The default text color (foreground brush).
public Brush Foreground { get; set; }

// The background.
public Brush Background { get; set; }

// The width of the line number. Default is 50.
public GridLength LineNumberWidth { get; set; }

// The style of the line number.
public Style LineNumberStyle { get; set; }

// The width of the change type symbol. Default is 20.
public GridLength ChangeTypeWidth { get; set; }

// The style of the change type symbol.
public Style ChangeTypeStyle { get; set; }

// The style of the text.
public Style TextStyle { get; set; }

// true if the text is selection enabled; otherwise, false. Default is true.
public bool IsTextSelectionEnabled { get; set; }

// true if collapse unchanged sections; otherwise, false. Default is false.
public bool IsUnchangedSectionCollapsed { get; set; }

// The lines for context. Default is 2.
public int LineCountForContext { get; set; }

// true if the file selector menu button is enabled; otherwise, false. Default is true.
public bool IsFileMenuEnabled { get; set; }

// The height of command bar. Default is 50.
public GridLength CommandBarHeight { get; set; }

// The default label position of command bar. Default is Right.
public CommandBarDefaultLabelPosition CommandLabelPosition { get; set; }

// The collection of secondary command elements for the command bar.
public IObservableVector<ICommandBarElement> SecondaryCommands { get; }
```

## WPF Controls

[![NuGet](https://img.shields.io/nuget/v/DiffPlex.Wpf.svg)](https://www.nuget.org/packages/DiffPlex.Wpf/)

DiffPlex WPF control library `DiffPlex.Wpf` is used to render textual diffs in your WPF application.
It targets `.NET 9`, `.NET 8`, `.NET 6`, `.NET Framework 4.8` and `.NET Framework 4.6`.

```csharp
using DiffPlex.Wpf.Controls;
```

To import the controls into your window/page/control, please insert following attribute into the root node (such as `<Window />`) of your xaml files.

```
xmlns:diffplex="clr-namespace:DiffPlex.Wpf.Controls;assembly=DiffPlex.Wpf"
```

- `DiffViewer` Textual diffs viewer control with view mode switching by setting an old text and a new text to diff.
- `SideBySideDiffViewer` Side-by-side (splitted) textual diffs viewer control by setting a diff model `SideBySideDiffModel`.
- `InlineDiffViewer` Inline textual diffs viewer control by setting a diff model `DiffPaneModel`.

For example.

```xaml
<diffplex:DiffViewer x:Name="DiffView" />
```

```csharp
DiffView.OldText = oldText;
DiffView.NewText = newText;
```

![WPF sample](./images/wpf_side_light.jpg)

You can also customize the style.
Following are some of the properties you can get or set.

```csharp
// The header of old text.
public string OldTextHeader { get; set; }

// The header of new text.
public string NewTextHeader { get; set; }

// true if it is in side-by-side (split) view;
// otherwise, false, in inline (unified) view.
public bool IsSideBySideViewMode { get; }

// true if collapse unchanged sections; otherwise, false.
public bool IgnoreUnchanged { get; set; }

// Hides the line numbers.
public bool HideLineNumbers { get; set; }

// The font size.
public double FontSize { get; set; }

// The preferred font family.
public FontFamily FontFamily { get; set; }

// The font weight.
public FontWeight FontWeight { get; set; }

// The font style.
public FontStyle FontStyle { get; set; }

// The font-stretching characteristics.
public FontStretch FontStretch { get; set; }

// The default text color (foreground brush).
public Brush Foreground { get; set; }

// The background brush of the line inserted.
public Brush InsertedBackground { get; set; }

// The background brush of the line deleted.
public Brush DeletedBackground { get; set; }

// The text color (foreground brush) of the line number.
public Brush LineNumberForeground { get; set; }

// The width of the line number and change type symbol.
public int LineNumberWidth { get; set; }

// The background brush of the line imaginary.
public Brush ImaginaryBackground { get; set; }

// The text color (foreground brush) of the change type symbol.
public Brush ChangeTypeForeground { get; set; }

// The background brush of the header.
public Brush HeaderBackground { get; set; }

// The height of the header.
public double HeaderHeight { get; set; }

// The background brush of the grid splitter.
public Brush SplitterBackground { get; set; }

// The width of the grid splitter.
public Thickness SplitterWidth { get; set; }

// A value that represents the actual calculated width of the left side panel.
public double LeftSideActualWidth { get; }

// A value that represents the actual calculated width of the right side panel.
public double RightSideActualWidth { get; }
```

And you can listen following event handlers.

```csharp

// Occurs when the grid splitter loses mouse capture.
public event DragCompletedEventHandler SplitterDragCompleted;

// Occurs one or more times as the mouse changes position when the grid splitter has logical focus and mouse capture.
public event DragDeltaEventHandler SplitterDragDelta;

// Occurs when the grid splitter receives logical focus and mouse capture.
public event DragStartedEventHandler SplitterDragStarted;

// Occurs when the view mode is changed.
public event EventHandler<ViewModeChangedEventArgs> ViewModeChanged;
```

## WinForms Controls

[![NuGet](https://img.shields.io/nuget/v/DiffPlex.Wpf.svg)](https://www.nuget.org/packages/DiffPlex.Wpf/)

Windows Forms control of diff viewer is a WPF element host control.
It is also included in `DiffPlex.Wpf` assembly.
You can import it to use in your Windows Forms application.
It targets `.NET 8`, `.NET 6`, `.NET Framework 4.8` and `.NET Framework 4.6`.

```csharp
using DiffPlex.WindowsForms.Controls;
```

Then you can add the following control in window or user control.

- `DiffViewer` Textual diffs viewer control with view mode switching by setting an old text and a new text to diff.

For example.

```csharp
public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        var diffView = new DiffViewer
        {
            Margin = new Padding(0),
            Dock = DockStyle.Fill,
            OldText = oldText,
            NewText = newText
        };
        Controls.Add(diffView);
    }
}
```

![Windows Forms sample](./images/wpf_side_light.jpg)

You can also customize the style.
Following are some of the properties you can get or set.

```csharp
// The header of old text.
public string OldTextHeader { get; set; }

// The header of new text.
public string NewTextHeader { get; set; }

// true if it is in side-by-side (split) view;
// otherwise, false, in inline (unified) view.
public bool IsSideBySideViewMode { get; }

// true if collapse unchanged sections; otherwise, false.
public bool IgnoreUnchanged { get; set; }

// The font size.
public double FontSize { get; set; }

// The preferred font family names in string.
public string FontFamilyNames { get; set; }

// The font weight.
public int FontWeight { get; set; }

// The font style.
public bool IsFontItalic { get; set; }

// The default text color (foreground brush).
public Color ForeColor { get; set; }

// The background brush of the line inserted.
public Color InsertedBackColor { get; set; }

// The background brush of the line deleted.
public Color DeletedBackColor { get; set; }

// The text color (foreground color) of the line number.
public Color LineNumberForeColor { get; set; }

// The width of the line number and change type symbol.
public int LineNumberWidth { get; set; }

// The background brush of the line imaginary.
public Color ImaginaryBackColor { get; set; }

// The text color (foreground color) of the change type symbol.
public Color ChangeTypeForeColor { get; set; }

// The background brush of the header.
public Color HeaderBackColor { get; set; }

// The height of the header.
public double HeaderHeight { get; set; }

// The background brush of the grid splitter.
public Color SplitterBackColor { get; set; }

// The width of the grid splitter.
public Padding SplitterWidth { get; set; }

// A value that represents the actual calculated width of the left side panel.
public double LeftSideActualWidth { get; }

// A value that represents the actual calculated width of the right side panel.
public double RightSideActualWidth { get; }
```
