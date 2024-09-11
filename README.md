Using `wxFileHistory` to build a list of recently opened files for C++ Desktop Apps.

The code in this repository shows how to integrate `wxFileHistory` with `wxWidgets`' Document-View Framework. If you want an example on how to use `wxFileHistory` manually, then scroll to the bottom of this page.

[![Video](/output.gif)](https://youtu.be/u3Y5qkJob04)

Full Tutorial: https://youtu.be/u3Y5qkJob04

This tutorial is a part of the series about building a multiplatform Paint App in C++ with wxWidgets: https://www.youtube.com/watch?v=Spt5VF1aSps&list=PL0qQTroQZs5sxKZDdJrn8fEjNXCPT7f5T

## Requirements

This works on Windows, Mac, and Linux. You'll need `cmake` and a C++ compiler (tested on `clang`, `gcc`, and MSVC).

Linux builds require the GTK3 library and headers installed in the system.

## Building

To build the project, use:

```bash
cmake -S. -Bbuild
cmake --build build
```

This will create a directory named `build` and create all build artifacts there. The main executable can be found in the `build/subprojects/Build/wx_filehistory_tutorial_core` folder.

## Simple Example

Here's the `wxFileHistory` example from the beginning of the video tutorial:

```cpp
#include <wx/wx.h>
#include <wx/filedlg.h>
#include <wx/filehistory.h>
#include <wx/config.h>

class MyApp : public wxApp
{
public:
    virtual bool OnInit();
};

class MyFrame : public wxFrame
{
public:
    MyFrame(const wxString &title, const wxPoint &pos, const wxSize &size);
    virtual ~MyFrame()
    {
    }

private:
    void SetupMenuBar();

    wxFileHistory fileHistory;
};

wxIMPLEMENT_APP(MyApp);

bool MyApp::OnInit()
{
    SetAppName("FileHistoryApp");

    MyFrame *frame = new MyFrame("Hello World", wxDefaultPosition, wxDefaultSize);
    frame->Show(true);
    return true;
}

MyFrame::MyFrame(const wxString &title, const wxPoint &pos, const wxSize &size)
    : wxFrame(NULL, wxID_ANY, title, pos, size)
{
    SetupMenuBar();
}

void MyFrame::SetupMenuBar()
{
    constexpr int ClearHistoryMenuId = 1000;

    wxMenu *menuFile = new wxMenu;
    menuFile->Append(wxID_NEW);
    menuFile->Append(wxID_OPEN);
    menuFile->AppendSeparator();

    wxMenu *historySubMenu = new wxMenu;
    historySubMenu->Append(ClearHistoryMenuId, "Clear history");

    menuFile->AppendSubMenu(historySubMenu, "Open Recent");
    menuFile->Append(wxID_EXIT);

    this->Bind(
        wxEVT_MENU, [&](wxCommandEvent &event)
        {
            while(fileHistory.GetCount() > 0) {
                fileHistory.RemoveFileFromHistory(0);
            }
            fileHistory.Save(*wxConfig::Get()); },
        ClearHistoryMenuId);

    this->Bind(
        wxEVT_MENU, [&](wxCommandEvent &event)
        {
            wxFileDialog openFileDialog(this, "Open file", "", "", "All files (*.*)|*.*",
                                        wxFD_OPEN | wxFD_FILE_MUST_EXIST);
            if (openFileDialog.ShowModal() == wxID_OK)
            {
                wxString path = openFileDialog.GetPath();
                wxMessageBox("Opening: " + path);

                fileHistory.AddFileToHistory(path);
                fileHistory.Save(*wxConfig::Get());
            } },
        wxID_OPEN);

    this->Bind(
        wxEVT_MENU, [&](wxCommandEvent &event)
        { Close(); },
        wxID_EXIT);

    fileHistory.UseMenu(historySubMenu);
    fileHistory.Load(*wxConfig::Get());

    this->Bind(
        wxEVT_MENU, [&](wxCommandEvent &event)
        {
            const auto historyId = event.GetId() - fileHistory.GetBaseId();
            const auto path = fileHistory.GetHistoryFile(historyId);
            wxMessageBox("You clicked: " + path); },
        fileHistory.GetBaseId(),
        fileHistory.GetBaseId() + fileHistory.GetMaxFiles());

    wxMenu *editMenu = new wxMenu();
    editMenu->Append(wxID_COPY);
    editMenu->Append(wxID_CUT);
    editMenu->Append(wxID_PASTE);

    wxMenu *menuHelp = new wxMenu;
    menuHelp->Append(wxID_ABOUT);
    wxMenuBar *menuBar = new wxMenuBar;

    menuBar->Append(menuFile, "&File");
    menuBar->Append(editMenu, "Edit");
    menuBar->Append(menuHelp, "&Help");

    SetMenuBar(menuBar);
}
```

---
Check out the blog for more! [www.lukesdevtutorials.com](https://www.lukesdevtutorials.com)
---

