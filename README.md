# Лабораторная работа №4. Растеризация окружностей

**Дисциплина:** Компьютерная графика
**Среда разработки:** Lazarus IDE (Free Pascal Compiler)
**Вариант 1:** Вывести вписанную и описанную окружности квадрата по его длине стороны.

## Задание

1. Написать программу, реализующую алгоритмы растеризации окружности с их последующей прорисовкой. Координаты центра и радиуса задаются пользователем. Реализовать:
   - построение окружности по уравнению окружности;
   - построение окружности по параметрическому уравнению;
   - алгоритм Брезенхема для построения окружности;
   - встроенные средства языка программирования.
2. Проиллюстрировать разницу между результатами или её отсутствие.
3. Ответить на контрольные вопросы.
4. Оформить отчёт.

По индивидуальному варианту: по заданной длине стороны квадрата `a` вычисляются радиус вписанной окружности (`R = a/2`) и радиус описанной окружности (`R = a*√2/2`), после чего обе окружности строятся всеми четырьмя способами.

## Программа

Форма содержит:

| Компонент | Имя | Назначение |
|---|---|---|
| TLabel | `Label1` | подпись "Длина стороны квадрата a:" |
| TEdit | `EditSide` | ввод длины стороны квадрата |
| TButton | `ButtonBuild` | кнопка "Построить" |
| TButton | `ButtonSave` | кнопка "Сохранить" |
| TImage | `Image1` | отображение результата |
| TSavePictureDialog | `SavePictureDialog1` | диалог сохранения файла |

```pascal
unit Unit1;

{$mode objfpc}{$H+}

interface

uses
  Classes, SysUtils, Forms, Controls, Graphics, Dialogs, ExtDlgs, ExtCtrls,
  StdCtrls, LCLIntf, LCLType, Math;

type

  { TForm1 }

  TForm1 = class(TForm)
    ButtonBuild: TButton;
    ButtonSave: TButton;
    EditSide: TEdit;
    Image1: TImage;
    Label1: TLabel;
    SavePictureDialog1: TSavePictureDialog;
    procedure ButtonBuildClick(Sender: TObject);
    procedure ButtonSaveClick(Sender: TObject);
  private
  public
  end;

var
  Form1: TForm1;

implementation

{$R *.lfm}

{ TForm1 }

// ---------------------------------------------------------------------
// 1. Растеризация окружности по уравнению окружности x^2+y^2=R^2
//    (строим 1/8 окружности и зеркально отражаем на остальные 7 октантов)
// ---------------------------------------------------------------------
procedure CircleEquation(x0, y0, r: Integer; canvas: TCanvas);
var
  x, y: Integer;
begin
  x := 0;
  while x <= round(r / sqrt(2)) do
  begin
    y := round(sqrt(sqr(r) - sqr(x)));
    canvas.Pixels[x0 + x, y0 + y] := canvas.Pen.Color;
    canvas.Pixels[x0 + x, y0 - y] := canvas.Pen.Color;
    canvas.Pixels[x0 - x, y0 + y] := canvas.Pen.Color;
    canvas.Pixels[x0 - x, y0 - y] := canvas.Pen.Color;
    canvas.Pixels[x0 + y, y0 + x] := canvas.Pen.Color;
    canvas.Pixels[x0 + y, y0 - x] := canvas.Pen.Color;
    canvas.Pixels[x0 - y, y0 + x] := canvas.Pen.Color;
    canvas.Pixels[x0 - y, y0 - x] := canvas.Pen.Color;
    x := x + 1;
  end;
end;

// ---------------------------------------------------------------------
// 2. Растеризация окружности по параметрическому уравнению
//    x = x0+R*cos(tau), y = y0+R*sin(tau), шаг tau = 1/R
// ---------------------------------------------------------------------
procedure CircleParametric(x0, y0, r: Integer; canvas: TCanvas);
var
  tau, tau0: Real;
  x, y: Integer;
begin
  tau0 := 1 / r;
  tau := 0;
  while tau < 2 * pi do
  begin
    x := round(x0 + r * cos(tau));
    y := round(y0 + r * sin(tau));
    canvas.Pixels[x, y] := canvas.Pen.Color;
    tau := tau + tau0;
  end;
end;

// ---------------------------------------------------------------------
// 3. Растеризация окружности по алгоритму Брезенхема
//    (строится 1/8 окружности в первом октанте, остальные 7 - симметрично)
// ---------------------------------------------------------------------
procedure CircleBresenham(x0, y0, r: Integer; canvas: TCanvas);
var
  x, y, delta, d1, d2: Integer;
begin
  x := 0;
  y := r;
  delta := 2 - 2 * r;
  while x <= y do
  begin
    // вывод текущего пикселя и семи симметричных ему
    canvas.Pixels[x0 + x, y0 + y] := canvas.Pen.Color;
    canvas.Pixels[x0 + y, y0 + x] := canvas.Pen.Color;
    canvas.Pixels[x0 + y, y0 - x] := canvas.Pen.Color;
    canvas.Pixels[x0 + x, y0 - y] := canvas.Pen.Color;
    canvas.Pixels[x0 - x, y0 - y] := canvas.Pen.Color;
    canvas.Pixels[x0 - y, y0 - x] := canvas.Pen.Color;
    canvas.Pixels[x0 - y, y0 + x] := canvas.Pen.Color;
    canvas.Pixels[x0 - x, y0 + y] := canvas.Pen.Color;

    d1 := 2 * (delta + y) - 1;
    d2 := 2 * (delta - x) - 1;

    if (delta < 0) and (d1 <= 0) then
    begin
      // горизонтальный шаг
      x := x + 1;
      delta := delta + 2 * x + 1;
    end
    else if (delta > 0) and (d2 > 0) then
    begin
      // вертикальный шаг
      y := y - 1;
      delta := delta - 2 * y + 1;
    end
    else
    begin
      // диагональный шаг
      x := x + 1;
      y := y - 1;
      delta := delta + 2 * (x - y);
    end;
  end;
end;

// ---------------------------------------------------------------------
// 4. Растеризация окружности встроенными средствами (TCanvas.Ellipse)
// ---------------------------------------------------------------------
procedure CircleBuiltIn(x0, y0, r: Integer; canvas: TCanvas);
begin
  canvas.Brush.Style := bsClear;
  canvas.Ellipse(x0 - r, y0 - r, x0 + r, y0 + r);
end;

// ---------------------------------------------------------------------
// Кнопка "Построить": по заданной длине стороны квадрата a строит
// сам квадрат, его вписанную окружность (R=a/2) и описанную
// окружность (R=a*sqrt(2)/2), каждую - четырьмя способами разным
// цветом, чтобы проиллюстрировать совпадение/расхождение результатов.
// ---------------------------------------------------------------------
procedure TForm1.ButtonBuildClick(Sender: TObject);
var
  a, border, size, cx, cy: Integer;
  rIn, rOut: Integer;
begin
  if not TryStrToInt(Trim(EditSide.Text), a) or (a <= 0) then
  begin
    ShowMessage('Введите длину стороны квадрата - положительное целое число!');
    Exit;
  end;

  rIn := a div 2;                      // радиус вписанной окружности
  rOut := round(a * sqrt(2) / 2);      // радиус описанной окружности

  border := 40;
  size := 2 * rOut + 2 * border;
  cx := size div 2;
  cy := size div 2;

  Image1.Picture.Bitmap.PixelFormat := pf24bit;
  Image1.Picture.Bitmap.SetSize(size, size);

  with Image1.Picture.Bitmap.Canvas do
  begin
    Brush.Style := bsSolid;
    Brush.Color := RGB(255, 255, 255);
    FillRect(Rect(0, 0, size, size));

    // квадрат - серым
    Brush.Style := bsClear;
    Pen.Color := RGB(160, 160, 160);
    Rectangle(cx - a div 2, cy - a div 2, cx + a div 2, cy + a div 2);
  end;

  // вписанная окружность (R = a/2): по уравнению - синим,
  // по параметрическому уравнению - пурпурным,
  // по Брезенхему - зелёным,
  // встроенными средствами - чёрным
  Image1.Picture.Bitmap.Canvas.Pen.Color := RGB(0, 0, 255);
  CircleEquation(cx, cy, rIn, Image1.Picture.Bitmap.Canvas);
  Image1.Picture.Bitmap.Canvas.Pen.Color := RGB(255, 0, 255);
  CircleParametric(cx, cy, rIn, Image1.Picture.Bitmap.Canvas);
  Image1.Picture.Bitmap.Canvas.Pen.Color := RGB(0, 180, 0);
  CircleBresenham(cx, cy, rIn, Image1.Picture.Bitmap.Canvas);
  Image1.Picture.Bitmap.Canvas.Pen.Color := RGB(0, 0, 0);
  CircleBuiltIn(cx, cy, rIn, Image1.Picture.Bitmap.Canvas);

  // описанная окружность (R = a*sqrt(2)/2) - теми же 4 способами/цветами
  Image1.Picture.Bitmap.Canvas.Pen.Color := RGB(0, 0, 255);
  CircleEquation(cx, cy, rOut, Image1.Picture.Bitmap.Canvas);
  Image1.Picture.Bitmap.Canvas.Pen.Color := RGB(255, 0, 255);
  CircleParametric(cx, cy, rOut, Image1.Picture.Bitmap.Canvas);
  Image1.Picture.Bitmap.Canvas.Pen.Color := RGB(0, 180, 0);
  CircleBresenham(cx, cy, rOut, Image1.Picture.Bitmap.Canvas);
  Image1.Picture.Bitmap.Canvas.Pen.Color := RGB(0, 0, 0);
  CircleBuiltIn(cx, cy, rOut, Image1.Picture.Bitmap.Canvas);

  Image1.Stretch := True;
  Image1.Proportional := True;
  Image1.Invalidate;
end;

procedure TForm1.ButtonSaveClick(Sender: TObject);
begin
  if SavePictureDialog1.Execute then
    Image1.Picture.SaveToFile(SavePictureDialog1.FileName);
end;

end.
```

## Результат работы программы

*(скриншоты добавить в папку `screenshots/`: стартовое окно, окно с введённой длиной стороны, построенный результат - квадрат и обе окружности четырьмя способами, сохранение файла)*

## Как запустить

1. Установить Lazarus IDE (Free Pascal Compiler).
2. Создать новый проект "Application".
3. Разместить на форме компоненты согласно таблице выше (`EditSide`, `ButtonBuild`, `ButtonSave`, `Image1`, `SavePictureDialog1`), привязать обработчики `OnClick`.
4. Заменить содержимое `Unit1.pas` кодом из этого репозитория.
5. На macOS при ошибке линковщика `ld: malformed method list atom` добавить в Project Options → Compiler Options → Custom Options параметр `-WM10.9`.
6. Запустить (F9), ввести длину стороны квадрата, нажать "Построить", затем "Сохранить".

## Вывод

Все четыре способа растеризации окружности (по уравнению окружности, по параметрическому уравнению, по алгоритму Брезенхема и встроенными средствами `TCanvas.Ellipse`) дают визуально совпадающий результат - окружности ложатся друг на друга. Различия, если они проявляются, заметны лишь в отдельных пикселях на границе окружности и связаны с округлением вещественных координат до целых значений пикселей.
