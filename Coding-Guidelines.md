# x64dbg Coding Guidelines #

v1.05

---

## Introduction

x64dbg has become quite a big project and therefore it is wise to enforce certain coding style parts, so everyone is writing code in an equal manner. After publication of this document every piece of code written for x64dbg **should** follow these guidelines, please keep this in mind while writing your code.

###A General Rule
When you edit existing code, try to follow the style used in that particular file, refactor the whole code if you are certain it does not comply with the coding guidelines in this document.

Another thing that is very important is: **use common sense**. If you cannot find a convention, just see the context and try to mimic that. Please inform me about this though, maybe there is something missing in this document.

###Another Thing
Specifically for x64dbg I coded [AStyleWhore](https://bitbucket.org/mrexodia/astylewhore/), a tool to auto-format the source code in C/C++ projects. Do not worry about code formatting. It will be silently done in a pre-commit hook (run *install.bat* after cloning the project), so do not be surprised if your code looks good after you commit. It is the documented behavior.

## Indentation

Use 4 spaces to indent every new scope, notice that Visual Studio uses tabs per default, change this to 4 spaces before you start working.

Example:
```
CMDRESULT cbCreateThread(int argc, char* argv[])
{
	if(argc < 2)
	{
		dputs("not enough arguments!");
		return STATUS_ERROR;
	}
	uint address = 0;
	if(!valfromstring(argv[1], &address, false))
		return STATUS_ERROR;
	threadcreate(address);
	return STATUS_CONTINUE;
}
```

##Newlines
Some people tend to write as much code as possible on a single line. Avoid this and use newlines without hesitation!

Example Header (bad):
```
#ifndef CLOSEDIALOG_H
#define CLOSEDIALOG_H
#include <QDialog>
namespace Ui { class CloseDialog; }
class CloseDialog : public QDialog
{
    Q_OBJECT
public:
    explicit CloseDialog(QWidget *parent = 0);
    ~CloseDialog();
private:
    Ui::CloseDialog *ui;
};
#endif // CLOSEDIALOG_H
```

Example Header (correct):
```
#ifndef CLOSEDIALOG_H
#define CLOSEDIALOG_H

#include <QDialog>

namespace Ui
{
class CloseDialog;
}

class CloseDialog : public QDialog
{
    Q_OBJECT

public:
    explicit CloseDialog(QWidget *parent = 0);
    ~CloseDialog();
                       //dont't forget this newline!
private:
    Ui::CloseDialog *ui;
};

#endif // CLOSEDIALOG_H
```

Example Source Snippet (bad):
```
void ThreadView::contextMenuSlot(const QPoint & pos)
{
    foreach(QAction* action, mSetPriority->actions())
    {
        action->setCheckable(true);
        action->setChecked(false);
    }
    QString priority = getCellContent(getInitialSelection(), 6);
    if(priority == "Normal") mSetPriorityNormal->setChecked(true);
    else if(priority == "AboveNormal") mSetPriorityAboveNormal->setChecked(true);
    else if(priority == "TimeCritical") mSetPriorityTimeCritical ->setChecked(true);
    else if(priority == "Idle") mSetPriorityIdle->setChecked(true);
    else if(priority == "BelowNormal") mSetPriorityBelowNormal->setChecked(true);
    else if(priority == "Highest") mSetPriorityHighest->setChecked(true);
    else if(priority == "Lowest") mSetPriorityLowest->setChecked(true);
    wMenu->exec(mapToGlobal(pos)); //execute context menu
}
void ThreadView::SuspendThread()
{
    QString threadId=getCellContent(getInitialSelection(), 1);
    DbgCmdExecDirect(QString("suspendthread " + threadId).toUtf8().constData());
}
```

Example Source Snippet (correct):
```
void ThreadView::contextMenuSlot(const QPoint & pos)
{
    foreach(QAction* action, mSetPriority->actions())
    {
        action->setCheckable(true);
        action->setChecked(false);
    }

    QString priority = getCellContent(getInitialSelection(), 6);
    if(priority == "Normal")
        mSetPriorityNormal->setChecked(true);
    else if(priority == "AboveNormal")
        mSetPriorityAboveNormal->setChecked(true);
    else if(priority == "TimeCritical")
        mSetPriorityTimeCritical ->setChecked(true);
    else if(priority == "Idle")
        mSetPriorityIdle->setChecked(true);
    else if(priority == "BelowNormal")
        mSetPriorityBelowNormal->setChecked(true);
    else if(priority == "Highest")
        mSetPriorityHighest->setChecked(true);
    else if(priority == "Lowest")
        mSetPriorityLowest->setChecked(true);
 //newline here
    wMenu->exec(mapToGlobal(pos)); //execute context menu
}

void ThreadView::SuspendThread()
{
    QString threadId=getCellContent(getInitialSelection(), 1);
    DbgCmdExecDirect(QString("suspendthread " + threadId).toUtf8().constData());
}
```

## Internal Types
x64\_dbg has various internal types, use them!

Example:
```
//DBG
uint addr1 = GetContextData(UE_CIP);
//Bridge
duint addr2 = GetContextData(UE_CSP);
//GUI
int_t addr3 = rvaToVa(getInitialSelection());
uint_t addr4 = (uint_t)param1;

```

##Memory Allocation
x64\_dbg has it's own memory allocation class. If you need to allcoate memory, use the class called *Memory*. If you need to interchange data between the GUI and the DBG, use the *BridgeAlloc* and *BridgeFree* functions.

Example of using the *Memory* class:
```
void formathex(char* string)
{
    int len = (int)strlen(string);
    _strupr(string);

    //create a new char pointer with size 'len + 1' in bytes
    Memory<char*> new_string(len + 1, "formathex:new_string");

    //typecasting is done implicitly
    int* ptr = new_string;

    memset(new_string, 0, len + 1);
    for(int i = 0, j = 0; i < len; i++)
        if(isxdigit(string[i]))
            j += sprintf(new_string + j, "%c", string[i]);
    strcpy(string, new_string);

    //forget about cleaning up, the destructor will take care of this
}
```

##Spacing
Spaces are important for the readability of the code, as long as you do not overspace them. See the examples for an explanation.

Examples:
```
uint val=getvaluefromthread(hThread); //incorrect, very dense.
uint val = getvaluefromthread(hThread); //correct and more readable.
```

```
//clear example of overspacing.
if ( getvaluefromthread( hThread, 1 ) == secret ( 1, 2 ) )
    stopdebug ( );

//correct example, clean and readable.
if(getvaluefromthread(hThread, 1) == secret(1, 2))
    stopdebug();
```

##Variable Naming
In general, variable names should be descriptive and simple. Naming conventions differ in the various modules, so each module will have a small sub-header. **Avoid** using underscores in your variables, as it will make your code very bloated.

###Bridge
In short: no naming conventions, use anything you like.

###DBG
At the moment (8/1/2014) there are no real conventions, just go with camel case, first part lower.

Example:
```
HANDLE hThread;
BREAKPOINT curBp;
int count;
bool bThisIsATest;
```

###GUI
Basically the same as the DBG, but class variables are prefixed with 'm'.

Example:
```
class Test
{
public:
    Test();

private:
    QAction mTestAction;
    int mCip;
}
```

##Variable Declarations
We use a C++ compiler, so make the scope of your variables as small as possible.

Example (bad):
```
int Bridge::getValue(const char* szFilePath)
{
    int i = 0;
    int j = 0;
    int k = -1;
    char szDebugString[65536] = "";
    if(!DbgIsDebugging())
    {
		strcpy(szDebugString, "[ERROR] Not debugging!");
		return k;
	}
    if(someStatement)
    {
		for(i = 0; i < strlen(szFilePath); i++)
		{
			for(j = 0; j < 12; j++)
			{
				if(anotherFunction(szFilePath, i, j))
				{
					k = j;
					break;
				}
			}
		}
	}
	else
	{
		strcpy(szDebugString, "Some debug string...");
		return k;
	}
	return k;
}
```

Example (correct):
```
int Bridge::getValue(const char* szFilePath)
{
    int k = -1;
    if(!DbgIsDebugging())
		return k;
	char szDebugString[65536] = "";
    if(someStatement)
    {
		for(int i = 0; i < strlen(szFilePath); i++)
		{
			for(int j = 0; j < 12; j++)
			{
				if(anotherFunction(szFilePath, i, j))
				{
					k = j;
					break;
				}
			}
		}
	}
	else
	{
		strcpy(szDebugString, "Some debug string...");
		return k;
	}
	return k;
}
```

##Function Naming
Function naming conventions currently differ inside the various modules, so each module will have a small sub-header. The only overall convention is that you should not use underscores in your function names, unless this is specified in the convention (like *"\_dbg\_"*).

###Bridge
The bridge has various naming conventions for different types of functions.

#####Exports
Bridge exports have the WinAPI conventions: CamelCaseFunctions.

Example:
```
BRIDGE_IMPEXP const char* BridgeInit();
BRIDGE_IMPEXP const char* BridgeStart();
BRIDGE_IMPEXP void* BridgeAlloc(size_t size);
BRIDGE_IMPEXP void BridgeFree(void* ptr);
```

#####Others
Other functions follow the DBG variable naming conventions.

###DBG
The debugger has quite complex naming conventions.

#####Exports
DBG exports **must** start with *"\_dbg\_"* or *"\_plugin\_"* and they **must** be lower case.

Example:
```
//Some 'normal' exports.
DLL_EXPORT duint _dbg_memfindbaseaddr(duint addr, duint* size);
DLL_EXPORT bool _dbg_memread(duint addr, unsigned char* dest, duint size, duint* read);
DLL_EXPORT bool _dbg_memwrite(duint addr, const unsigned char* src, duint size, duint* written);
DLL_EXPORT bool _dbg_memmap(MEMMAP* memmap);

//Some plugin exports.
PLUG_IMPEXP void _plugin_logprintf(const char* format, ...);
PLUG_IMPEXP void _plugin_logputs(const char* text);
PLUG_IMPEXP void _plugin_debugpause();
PLUG_IMPEXP void _plugin_debugskipexceptions(bool skip);
```

#####Static functions
Static functions do not have conventions, since other files cannot see them, try to make up a sensible name though.

#####Command Callbacks
Command callbacks **must** be prefixed with *"cb"* and from there they follow camel case. They must also describe their type.

Example:
```
//Some debug command callbacks.
CMDRESULT cbDebugInit(int argc, char* argv[]);
CMDRESULT cbStopDebug(int argc, char* argv[]);
CMDRESULT cbDebugRun(int argc, char* argv[]);
CMDRESULT cbDebugErun(int argc, char* argv[]);

//Some general purpose instructions.
CMDRESULT cbInstrCmp(int argc, char* argv[]);
CMDRESULT cbInstrGpa(int argc, char* argv[]);
CMDRESULT cbInstrAdd(int argc, char* argv[]);
CMDRESULT cbInstrAnd(int argc, char* argv[]);
```

#####Module functions
Functions that could have a (static) class wrapped around it (still to be done) should be prefixed with their class name and they **must** be lowercase. The only exception to the prefix is when a function is used really often and a prefix would only require more typing.

Example:
```
//Script functions.
void scriptload(const char* filename);
void scriptunload();
void scriptrun(int destline);
void scriptstep();

//Breakpoint functions.
void bptobridge(const BREAKPOINT* bp, BRIDGEBP* bridge);
void bpcachesave(JSON root);
void bpcacheload(JSON root);
void bpclear();
```

When wrapped in a (static) class, just use Class::functionName, where the current function name would be "classfunctionname".

#####Anything else
Just follow the DBG variable naming conventions.

###GUI
The GUI has more simple naming conventions.

#####Exports
GUI exports have the same conventions as the DBG, but then the prefix is *"\_gui\_"*

Example (and actually the only two exports):
```
extern "C" __declspec(dllexport) int _gui_guiinit(int argc, char *argv[]);
extern "C" __declspec(dllexport) void* _gui_sendmessage(GUIMSG type, void* param1, void* param2);
```

#####Class functions
Go with the DBG variable naming conventions.

Example:
```
#ifndef SHORTCUTEDIT_H
#define SHORTCUTEDIT_H

#include <QLineEdit>

class ShortcutEdit : public QLineEdit
{
    Q_OBJECT
    QKeySequence key;
    int keyInt;

public:
    explicit ShortcutEdit(QWidget *parent = 0);
    const QKeySequence getKeysequence() const;

public slots:
    void setErrorState(bool error);

signals:
    void askForSave();

protected:
    void keyPressEvent(QKeyEvent* event);
};

#endif // SHORTCUTEDIT_H
```

#####Anything else
Just follow the DBG variable naming conventions.

##Struct, Enum, #define

###Bridge
Go for uppercase, for the outer name, anything you like for the inner names. Notice that bridgemain.h can be called from C, so keep this in mind when declaring the Structs and Enums.

Example:
```
//UPPERCASE, lowercase
typedef struct
{
    int count;
    MEMPAGE* page;
} MEMMAP;

//UPPERCASE, whatever you like
typedef struct
{
    int ThreadNumber;
    HANDLE hThread;
    DWORD dwThreadId;
    duint ThreadStartAddress;
    duint ThreadLocalBase;
    char threadName[MAX_THREAD_NAME_SIZE];
} THREADINFO;

#define MAX_LABEL_SIZE 256
#define MAX_COMMENT_SIZE 512
#define MAX_MODULE_SIZE 256
```

###DBG
Go for uppercase, for the outer name, anything you like for the inner names. Notice that _plugins.h can be called from C, so keep this in mind when declaring structs and Enums. See the Bridge for examples.

The only exception to this rule is inside classes. Follow the GUI conventions here.

###GUI
Inside classes, go for CamelCase. Otherwise, follow the DBG rules.

Example:
```
enum BeaTokenType
{
    //filling
    TokenComma,
    TokenSpace,
    TokenArgumentSpace,
    TokenMemoryOperatorSpace,
    //general instruction parts
    TokenPrefix,
    TokenUncategorized,
    TokenAddress, //jump/call destinations or displacements inside memory
    TokenValue,
    --cut from here--
};

struct BeaTokenValue
{
    int size; //value size
    int_t value; //value
};
```

##Headers and Source Files
Do not, I repeat: **DO NOT** slam your class into a single *.h* file! Only do this if you are working on a very simple class that is used as allocator or something, but in general: split header and source files. In short: put as much as possible in the source files.

Example (header):
```
#ifndef HISTORYLINEEDIT_H
#define HISTORYLINEEDIT_H

#include <QtGui>
#include <QLineEdit>

class HistoryLineEdit : public QLineEdit
{
    Q_OBJECT
public:
    explicit HistoryLineEdit(QWidget *parent = 0);
    void keyPressEvent(QKeyEvent* event);
    void addLineToHistory(QString parLine);
    void setFocus();

signals:
    void keyPressed(int parKey);

private:
    QList<QString> mCmdHistory;
    int mCmdIndex;
    bool bSixPressed;

};

#endif // HISTORYLINEEDIT_H
```

Example (source, code removed):
```
#include "HistoryLineEdit.h"
#include "Bridge.h"

HistoryLineEdit::HistoryLineEdit(QWidget *parent) : QLineEdit(parent)
{
}

void HistoryLineEdit::addLineToHistory(QString parLine)
{
}

void HistoryLineEdit::keyPressEvent(QKeyEvent* event)
{
}

void HistoryLineEdit::setFocus()
{
}
```

###An Important Rule
**DO NOT** include all the required *.h* files inside other header files. Only include a *.h* inside the header if it is absolutely required, otherwise include *.h* files inside your source file. This generally saves compile time and as the GUI now takes minutes to compile on a Dual Core this is important.

###Another Important Thing
Do not use ugly things like *"#pragma once"* inside your headers, just go with the #ifndef construction as seen in the example above.

##Final words
This should be the most important rules. I do not want to stretch out this document much more, but if there are any neccesary additions, feel free to contact me for a revision.

*Mr. eXoDia* (mr.exodia.tpodt@gmail.com)