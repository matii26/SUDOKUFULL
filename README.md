# SUDOKUFULL


MainWindow.xaml

<Window x:Class="Sudoku_HomeEdition.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:local="clr-namespace:Sudoku_HomeEdition"
    mc:Ignorable="d"
    MinHeight="320"
    MinWidth="620"
    WindowStartupLocation="CenterScreen"
    Title="Sudoku - Prepare" Height="450" Width="800">

    <StackPanel
        VerticalAlignment="Center"
        HorizontalAlignment="Center"
        Margin="50 30">
        <TextBlock
            TextAlignment="Center"
            FontSize="20">
            SUDOKU
        </TextBlock>
        <StackPanel
            Height="52"
            Orientation="Horizontal">
            <TextBlock 
                VerticalAlignment="Center"
                HorizontalAlignment="Center"
                TextAlignment="Right"
                Width="195"
                Padding="0 0 5 0"
                FontSize="16">Size of the board:</TextBlock>
            <Button x:Name="minusButton" Click="HandleMinusButtonClick" Height="32" Width="60" Padding="0 0 0 7" Margin="0">
                <Grid>
                    <TextBlock FontSize="25" Text="-" VerticalAlignment="Center" Height="32" LineHeight="32"/>
                </Grid>
            </Button>
            <TextBox 
                HorizontalContentAlignment="Center"
                VerticalContentAlignment="Center"
                VerticalAlignment="Center"
                Width="180"
                Height="32"
                FontSize="14"
                IsReadOnly="True"
                x:Name="boardSizeTextBox"></TextBox>
            <Button x:Name="plusButton" Click="HandlePlusButtonClick" Height="32" Width="60" Padding="0 0 0 7" Margin="0">
                
            </Button>
        </StackPanel>
        <StackPanel
            Margin="0 3 0 0"
            Height="52"
            Orientation="Horizontal">
           
            
        </StackPanel>
        <DockPanel
            Width="100"
            Height="50"
            Margin="0 30 0 0"
            HorizontalAlignment="Center"
            VerticalAlignment="Top">
            <Button
                FontSize="16"
                x:Name="playButton"
                Click="HandlePlayButtonClick">
                PLAY!
            </Button>
        </DockPanel>
    </StackPanel>
</Window>

MainWindow.xaml.cs

using System.Windows;

namespace Sudoku_HomeEdition
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>


    public partial class MainWindow : Window
    {
        private int boardSize = 0;
        
        public MainWindow()
        {
            InitializeComponent();
            InitializeAllProperties();
        }
        private void InitializeAllProperties()
        {
            SetBoardSize(3);
            
        }
        private void HandlePlayButtonClick(object sender, RoutedEventArgs e)
        {
            Game game = new Game();
            game.BoardSize = boardSize;
            
            Close();
            game.Show();
        }
        private void HandleMinusButtonClick(object sender, RoutedEventArgs e)
        {
            SetBoardSize(-1);
            
        }
        private void HandlePlusButtonClick(object sender, RoutedEventArgs e)
        {
            SetBoardSize(1);
            
        }
        private void SetBoardSize(int IncreaseOrSubtractBy)
        {
            if (boardSize > 1 || IncreaseOrSubtractBy > -1)
            {
                boardSize = boardSize + IncreaseOrSubtractBy;
                int boardSizeBySmallBoxes = boardSize * boardSize;
                boardSizeTextBox.Text = boardSizeBySmallBoxes.ToString() + " x " + boardSizeBySmallBoxes.ToString();
            }

        }
       
    }
}

Game.xaml

<Window x:Class="Sudoku_HomeEdition.Game"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Sudoku_HomeEdition"
        mc:Ignorable="d"
        WindowStartupLocation="CenterScreen"
        Title="Sudoku - Game" Height="1000" Width="1000">
    <Grid x:Name="table">

    </Grid>
</Window>


Game.xaml.cs

using System.Diagnostics;
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Controls.Primitives;
using System.Windows.Input;
using System.Windows.Media;
using static System.Formats.Asn1.AsnWriter;

namespace Sudoku_HomeEdition
{
    /// <summary>
    /// Logika interakcji dla klasy Game.xaml
    /// </summary>
    public partial class Game : Window
    {
        private int _boardSize;
        
        private TextBox[,,] boxDatabase;
        public int BoardSize
        {
            get { return _boardSize; }
            set { _boardSize = value; }
        }

        public Game()
        {
            InitializeComponent();
            Loaded += GameLoaded;
        }

        private void GameLoaded(object sender, RoutedEventArgs e)
        {
            Stopwatch stopwatch = new Stopwatch();
            stopwatch = Stopwatch.StartNew();
            InitializeGame(sender, e);
            stopwatch.Stop();
            Trace.WriteLine("Czas generowania planszy: " + stopwatch.Elapsed);
        }

        private void InitializeGame(object sender, RoutedEventArgs e)
        {
            int DBRow = 0;
            int DBColumn = 0;
            int DBDepth = 0;
            boxDatabase = new TextBox[_boardSize, _boardSize, _boardSize * _boardSize];
            Border boardBorder = new Border();
            boardBorder.BorderBrush = Brushes.Black;
            boardBorder.BorderThickness = new Thickness(3);
            UniformGrid board = new UniformGrid();
            board.Columns = _boardSize;
            board.Rows = _boardSize;
            boardBorder.Child = board;
            for (int i = 0; i < _boardSize * _boardSize; i++)
            {
                if (i % _boardSize == 0 && i != 0)
                {
                    DBRow++;
                    DBColumn = 0;
                }
                Border bigBoxBorder = new Border();
                bigBoxBorder.BorderBrush = Brushes.Black;
                bigBoxBorder.BorderThickness = new Thickness(3);
                UniformGrid bigBox = new UniformGrid();
                bigBox.Columns = _boardSize;
                bigBox.Rows = _boardSize;
                bigBoxBorder.Child = bigBox;
                for (int j = 0; j < _boardSize * _boardSize; j++)
                {
                    TextBox smallBox = new TextBox();
                    smallBox.HorizontalContentAlignment = HorizontalAlignment.Center;
                    smallBox.VerticalContentAlignment = VerticalAlignment.Center;
                    smallBox.TextChanged += HandleSmallBoxTextChanged;
                    bigBox.Children.Add(smallBox);
                    CommandManager.AddPreviewCanExecuteHandler(smallBox, DisableCopyPasteCut);
                    boxDatabase[DBRow, DBColumn, DBDepth] = smallBox;
                    DBDepth++;
                }
                DBDepth = 0;
                DBColumn++;
                board.Children.Add(bigBoxBorder);
            }
            table.Children.Add(boardBorder);
        }

        private void HandleSmallBoxTextChanged(object sender, TextChangedEventArgs e)
        {
            TextBox textBox = sender as TextBox;
            if (!string.IsNullOrEmpty(textBox.Text))
            {
                bool isTextBoxClean = IsTextBoxClean(textBox.Text);
                if (!isTextBoxClean)
                {
                    textBox.Text = RepairTextBox(textBox.Text);
                }
                CheckMyAnswers();
            }

        }
        private void CheckMyAnswers()
        {
            Stopwatch fullCheckTime = new Stopwatch();
            fullCheckTime.Start();
            Stopwatch stopwatch = new Stopwatch();
            bool isGameCompleted = true;
            stopwatch.Start();
            bool columnStatus = CheckColumns();
            stopwatch.Stop();
            Trace.WriteLine("columnsCheck time: " + stopwatch.Elapsed);
            if (!columnStatus)
            {
                isGameCompleted = false;
            }
            stopwatch.Reset();
            stopwatch.Start();
            bool rowStatus = CheckRows();
            stopwatch.Stop();
            Trace.WriteLine("rowsCheck time: " + stopwatch.Elapsed);
            if (!rowStatus)
            {
                isGameCompleted = false;
            }
            stopwatch.Reset();
            stopwatch.Start();
            bool bigBoxStatus = CheckBigBox();
            stopwatch.Stop();
            Trace.WriteLine("bigBoxCheck time: " + stopwatch.Elapsed);
            if (!bigBoxStatus)
            {
                isGameCompleted = false;
            }
            fullCheckTime.Stop();
            Trace.WriteLine("sudoku check time: " + fullCheckTime.Elapsed);
            fullCheckTime.Reset();
            Trace.WriteLine("");
            if (isGameCompleted)
            {
                Score scoreWindow = new Score();
                Close();
                scoreWindow.Show();
            }
        }
        private bool CheckColumns(int inBigBoxBoxPointer = 0)
        {
            bool isColumnsClear = true;
            List<string> columnValues = new List<string>();
            for (int column = 0; column < _boardSize; column++)
            {
                for (int row = 0; row < _boardSize; row++)
                {
                    for (int box = inBigBoxBoxPointer; box < _boardSize * _boardSize; box += _boardSize)
                    {
                        columnValues.Add(boxDatabase[row, column, box].Text);
                    }
                    for (int indexOfTheValueBeingChecked = 0; indexOfTheValueBeingChecked < columnValues.Count(); indexOfTheValueBeingChecked++)
                    {
                        string valueBeingChecked = columnValues[indexOfTheValueBeingChecked];
                        if (string.IsNullOrEmpty(valueBeingChecked))
                        {
                            isColumnsClear = false;
                            return isColumnsClear;
                        }
                        for (int listCheckerIterator = 0; listCheckerIterator < columnValues.Count(); listCheckerIterator++)
                        {
                            if (indexOfTheValueBeingChecked != listCheckerIterator)
                            {
                                if (valueBeingChecked == columnValues[listCheckerIterator])
                                {
                                    isColumnsClear = false;
                                    return isColumnsClear;
                                }
                            }
                        }
                    }
                    columnValues.Clear();
                }
            }
            if (inBigBoxBoxPointer < _boardSize)
            {
                CheckColumns(inBigBoxBoxPointer + 1);
            }
            return isColumnsClear;
        }
        private bool CheckRows()
        {
            bool isRowsClear = true;
            List<int> indexStarts = new List<int>();
            for (int i = 0; i < _boardSize * _boardSize; i++)
            {
                if (i % _boardSize == 0)
                {
                    indexStarts.Add(i);
                }
            }
            int boxIndex = 0;
            List<string> rowValuesList = new List<string>();
            for (int row = 0; row < _boardSize; row++)
            {
                for (int column = 0; column < _boardSize; column++)
                {
                    if (column != _boardSize - 1)
                    {
                        int bigBoxRowPointer = 0;
                        for (int x = 0; x < _boardSize; x++)
                        {
                            column = 0;
                            for (int throughBigBoxIterator = 0; throughBigBoxIterator < _boardSize; throughBigBoxIterator++)
                            {

                                for (int box = indexStarts[bigBoxRowPointer]; box < _boardSize + indexStarts[bigBoxRowPointer]; box++)
                                {
                                    string DBNumber = boxDatabase[row, column, box].Text;
                                    if (string.IsNullOrEmpty(DBNumber))
                                    {
                                        isRowsClear = false;
                                        return isRowsClear;
                                    }
                                    if (int.Parse(DBNumber) < 1 || int.Parse(DBNumber) > _boardSize * _boardSize)
                                    {
                                        isRowsClear = false;
                                        return isRowsClear;
                                    }
                                    boxIndex++;
                                    rowValuesList.Add(DBNumber);
                                    if (boxIndex == _boardSize * _boardSize)
                                    {
                                        boxIndex = 0;
                                        for (int indexOfTheValueBeingChecked = 0; indexOfTheValueBeingChecked < rowValuesList.Count; indexOfTheValueBeingChecked++)
                                        {
                                            for (int listCheckerIterator = 0; listCheckerIterator < rowValuesList.Count; listCheckerIterator++)
                                            {
                                                if (rowValuesList[indexOfTheValueBeingChecked] == rowValuesList[listCheckerIterator] && indexOfTheValueBeingChecked != listCheckerIterator)
                                                {
                                                    isRowsClear = false;
                                                    return isRowsClear;
                                                }
                                            }
                                        }
                                        rowValuesList.Clear();
                                    }
                                }
                                column++;

                            }
                            bigBoxRowPointer++;
                        }
                    }
                }
            }
            return isRowsClear;
        }
        private bool CheckBigBox()
        {
            bool isBigBoxClear = true;
            for (int i = 0; i < _boardSize; i++)
            {
                for (int j = 0; j < _boardSize; j++)
                {
                    for (int k = 0; k < _boardSize * _boardSize; k++)
                    {
                        string DBNumber = boxDatabase[i, j, k].Text;
                        if (string.IsNullOrEmpty(DBNumber))
                        {
                            isBigBoxClear = false;
                            return isBigBoxClear;
                        }
                        else
                        {
                            if (int.Parse(DBNumber) <= _boardSize * _boardSize && int.Parse(DBNumber) >= 1)
                            {
                                for (int l = 0; l < _boardSize * _boardSize; l++)
                                {
                                    if (l != k)
                                    {
                                        if (DBNumber == boxDatabase[i, j, l].Text)
                                        {
                                            isBigBoxClear = false;
                                            return isBigBoxClear;
                                        }
                                    }
                                }
                            }
                            else
                            {
                                isBigBoxClear = false;
                                return isBigBoxClear;
                            }
                        }
                    }
                }
            }
            return isBigBoxClear;

        }
        private bool IsTextBoxClean(string textToCheck)
        {
            Regex allowedCharacters = new Regex("[^0-9]");
            bool cleanStatus = true;
            if (allowedCharacters.IsMatch(textToCheck))
            {
                MessageBox.Show("Next time, try entering a number...");
                cleanStatus = false;
            }
            else
            {
                if (int.Parse(textToCheck) > _boardSize * _boardSize)
                {
                    MessageBox.Show("Whoah! Isn't this nuber too big?");
                    cleanStatus = false;
                }
                if (int.Parse(textToCheck) < 1)
                {
                    MessageBox.Show("Please, start from 1.");
                    cleanStatus = false;
                }
            }
            return cleanStatus;
        }
        private string RepairTextBox(string textToRepair)
        {
            Regex allowedCharacters = new Regex("[^1-9]");
            for (int i = 0; i < textToRepair.Length; i++)
            {
                if (allowedCharacters.IsMatch(textToRepair[i].ToString()))
                {
                    textToRepair = textToRepair.Remove(i, 1).Insert(i, "");
                }
            }
            return textToRepair;
        }
        private void DisableCopyPasteCut(object sender, CanExecuteRoutedEventArgs e)
        {
            if (e.Command == ApplicationCommands.Cut ||
                e.Command == ApplicationCommands.Copy ||
                e.Command == ApplicationCommands.Paste)
            {
                e.CanExecute = false;
                e.Handled = true;
            }
        }
    }
}


Score.xaml

<Window x:Class="Sudoku_HomeEdition.Score"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:Sudoku_HomeEdition"
        mc:Ignorable="d"
        WindowStartupLocation="CenterScreen"
        Title="Score" Height="450" Width="800">
    <Grid>
        <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center" FontSize="50" Text="You have Won!"></TextBlock>
    </Grid>
</Window>


Score.xaml.cs

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Shapes;

namespace Sudoku_HomeEdition
{
    /// <summary>
    /// Logika interakcji dla klasy Score.xaml
    /// </summary>
    public partial class Score : Window
    {
        public Score()
        {
            InitializeComponent();
        }
    }
}
