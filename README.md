# Asymptote tiny IDE
Asymptote: the Vector Graphics Language.

Asymptote is a powerful descriptive vector graphics language that provides a natural coordinate-based framework for technical drawing.
But, Asymptote is lack of an IDE, so this tiny IDE comes.

## Linux shell

Tools:
- inotifywait
- evince
- vscode

```shell
fname='1';
create_empty_eps(){
    # create an empty .eps file for evnice to open(auto-reload).
    echo '%!PS-Adobe-3.0 EPSF-3.0\n' >| ${fname}.eps;
}
create_empty_eps;

# start evince in background.
evince ${fname}.eps &;
evince_pid=$!;
# trap ctrl+c for killing background evnice process.
is_ctrl_c=0;
function ctrl_c() {
    echo "kill ${evince_pid}";
    kill -9 ${evince_pid};
    is_ctrl_c=1;
}
trap ctrl_c INT;

asy ${fname}.asy;
while [ ${is_ctrl_c} -ne 1 ];
do
    inotifywait -e CLOSE_WRITE ${fname}.asy;
    asy ${fname}.asy;
    if [ $? -ne 0 ]; then
        create_empty_eps;
    fi
done
```

## Windows PowerShell

Tools:
- https://github.com/thekid/inotify-win
- SumatraPDF
- vscode

```powershell
$fname='1'
function create_empty_eps{
    # create an empty .eps file for evnice to open(auto-reload).
    echo '%!PS-Adobe-3.0 EPSF-3.0\n' > "$fname.eps"
}
create_empty_eps

# start SumatraPDF.
# omit `.exe` of `SumatraPDF.exe` for Start-Process to process info.
$sumatra_app = Start-Process SumatraPDF "$fname.eps" -passthru
echo $sumatra_app

function run_asy{
    # `View output` is on by default on Windows, so use `-noV`
    # https://asymptote.sourceforge.io/FAQ/section9.html#psviewer
    &'asy.exe' -noV "$fname.asy"
}

run_asy
# trap ctrl+c for killing SumatraPDF process.
try {
    while($true){
        inotifywait.exe -e modify "$fname.asy"
        run_asy
        If ($LastExitCode -eq "1") {
            create_empty_eps
        }
    }
}
finally {
    $sumatra_app | Stop-Process
}
```
