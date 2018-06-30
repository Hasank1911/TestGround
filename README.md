# Things I like about the codebase

Apart from small bugs, system is bug-free.

General structure of the code is modular. It is ready for possible improvements.

Codebase is readable and each and every one of methods' task are understandable.

It is tested for various cases.

Use of Integer 2D array instead of int is clever. Primitive types would not allow the use of HashSets, and other object related work. And also we would have to assume that empty cells are marked as "0", which would be still okay, but having "null" is preferable. 

# Problems/suggested improvements

Below problems are sorted according to their priority. Problem with highest priority is listed first.

1 -) Performance of validity check in file BoardLogic.java

	Current system checks for validity of digits(if they are in range of 1-9), cells(3x3 pieces), columns and rows. 
	Validate column, row and cell method use validatePartial(Integer[] ints) method to ensure that there is no duplicate numbers.
	But this method is works very inefficiently. 
	Rather than using a hashset and come up with an O(n) algorithm, for each number, it checks all of the next numbers. 
	This leads to an O(n^2) algorithm. 
	Instead of this IsValid() algorithm, the proposed algorithm is; (Please note that we will also keep the list of coordinates that are erroneous)
	
	- have 3 arrays of hashmap (HashSet would be enough if we didn't have to keep the list of erroneous coordinates)
	- each array has 9 elements for 9x9 sudoku (9 rows, 9 columns, 9 cells)
	- we will visit each element on the board once, and check for digit validity, column, row and cell with using hashmaps in O(1)
	- Hashmap's key will be value of the board elements,  Hashmap's value will be list of coordinates that has this value.
	
	For example; 
	for row; 1 2 3 4 5 6 7 4 9 
	
	Hashmap has a entry = "4" -> ["3","7"] (key -> value  pair, value is the locations)
	
	- when we see a erroneous entry, we will store it into our list of errors along with the other locations that made this entry erroneous. We will use a hashSet for storing erroneous entries, so there can not be any duplicate items.
	
	- we can optimize the storing process like below;
	
		- if we see erroneous entry that was seen only once, we can try to add this location and previous location to our error list
		- if this erroneous entry was seen more than once before, that means we have already added the previous ones to our error list, so we only need to try to add this location to our error list.
	
	 
2 -) Always generating same puzzle.

	Game is always generating same puzzle in a static way. That means every customer will only see the same puzzle every time they browse the webpage. There can be two ways of solving this problem. 
	First one is generating a new valid sudoku puzzle whenever requested(procedurally generated). 
	But since that can be time consuming, the second approach would be to have multiple pre-prepared sudoku puzzles in store, and return one of them randomly whenever a request comes. 
	
	There are important things to keep in mind when creating a new sudoku puzzle. 
	First of all it should be valid (each row, column and cell should have distinct numbers). 
	It should be solvable, each location in board should have available number to put into them and each empty location's value should be deducible. 
	It should only have single solution. And we need to be careful about difficulty of the puzzle.
	
	Validity of the puzzle can be guaranteed with using HashSets. 
	Difficulty of the puzzle can be determined by number of deductions the person need to make in order to solve the puzzle. 
	There are different approaches in the literature. 
	Some of them fill the entire board first and remove elements if the board is still solvable. 
	Some of the approaches adds elements until the board is solvable.
	
3 -) Error list has duplicate items in file BoardLogic.java.

	Currently we are recording erroneous coordinates using a LinkedList. 
	Same coordinates i and j, can be erroneous for multiple reasons (row, column and cell) and we will have same coordinates up to three times in our list. 
	Instead of LinkedList, a HashSet can be used to solve this problem.
	
4 -) Error list has no "type of error" information in file BoardLogic.java. (related to problem 3)
	
	Currently error list is just a list of coordinates and no information about why they are erroneous is not attached to them. 
	We can use a HashMap to keep this list. Each coordinate (combination of i and j) will be the key, and value will be the list of reasons why they are erroneous. 
	This value can have up to three elements ("row", "column" and "cell").
	
5- ) shouldErrorOnEmptyBoardFields() method will definitely fail in BoardTests.java
	
	Because isValid() method does not check if all elements are null. 
	It simply skips null elements and works on actual integer values. and getBoardState(Board b) method does not do anything special if all elements are null. 
	We can add special case in IsValid method to fix this problem.
	
6 -) shouldErrorOnInvalidBoardRow() and shouldErrorOnInvalidBoardColumn() methods in file BoardTest.java

	These two methods test the system for row and column errors. 
	But it uses coordinates that are also in the same cells. 
	Program can have a working "cellCheck" method and its "rowCheck" and "columnCheck" methods may not be working. 
	In this case, program will recognize the cell error, raise the exception, and test method will see this exception and falsely decide that program is running correctly. 
	So this test method will fail. 
	
	Instead for  "shouldErrorOnInvalidBoardRow()" method;
	
	It should check for some coordinates like "fields[0][0] = 1; fields[9][0] = 1;". 
	Because they are not in the same cell, they are not in the same column, they are only in same row. 
	This test will only pass if and only if program can check row errors correctly.
	
	For  "shouldErrorOnInvalidBoardColumn()" method;
	
	It should check for "fields[0][0] = 1; fields[0][9] = 1;" for same reasons as row method.

7 -) "int cellCount = 3;" and "int length = 3;" statements in validateCell and ValidateCells method in file BoardLogic.java

	If we are going to only support 9x9 sudoku samples, these lines are not a problem. 
	But if we predict that we can add sudoku puzzles with different sizes in the future(like 4x4, 16x16,..), then these statements will cause an error. 
	Instead of using a magic number like 3, we can call Math.sqrt(sudokuSize) method.

8 -) Multiple occurences of "b.getFields()[i][j]" in file BoardLogic.java
	
	Currently when validity algorithms access the board values using "b.getFields()[i][j];" statement (i and j are the coordinates). 
	Instead of this, we can create a local variable local_board = b.getFields() and use this local variable as "local_board[i][j]".
	This way we won't have to call getFields() method whenever we need to access a value in the board.
	
[j]: coordinate
[i]: coordinate
	
9 - ) shouldHaveContent() method in file StaticBoardGeneratorTest.java	
	
	This part of the code has some performance issues;
	
	boolean hasContent = false;
	for (int i = 0; i < fields.length; ++i)
		for (int j = 0; j < fields[i].length; ++j)
			hasContent = hasContent || (fields[i][j] != null);
	assertTrue(hasContent);
		
	We should fix it like below;
	
	boolean hasContent = false;
	int length = fields.length;
	int rowLength = fields[0].length;
	for (int i = 0; i < length; ++i) {
		for (int j = 0; j < rowLength; ++j) {
			if(fields[i][j] != null) {
				hasContent = true;
				break;
			}
		}
		if(hasContent) {
		 	break;
		}
	}		
	assertTrue(hasContent);
	
	
	This way, whenever we see a non-null value, we will cease the search and use assert method and we will not keep searching unnecessarily. 
	Also we should keep "length" and "rowLength" local variables to increase the performance.

10 -) getBoardState(Board b) method in file BoardLogic.java
	
	There are two "for" statements "for(int i = 0; i < b.getFields().length; i++)" and "for (int j = 0; j < b.getFields().length; j++)". 
	We should create a local variable "int length = b.getFields().length;" and use it in "for" statement so we won't have to call getFields() method in every iteration of for loop. (as "i < length").
			
11 -) getBoardState(Board b) method in file BoardLogic.java
	
	This method has access to board object, it can update the board's state after finding it rather than leaving this update to other methods.
	
12 -) shouldBe9x9() method in file StaticBoardGeneratorTest.java	
	
	This test method checks for every rows' length. Since in java 2D arrays([][]) are rectangle shaped, we can check only for first row's length and confirm the 2D array's length;
	
	Integer[][] fields = b.getFields();
	assertThat(fields.length, is(9)); // we may have to avoid using magic number "9"
	assertThat(fields[0].length, is(9));
	

# Highest priority improvement

1 -) Performance of validity check in file BoardLogic.java 
	
	This problem has the highest priority because it has a very high impact on system's performance. 
	IsValid() method will probably be called several times, and having an inefficient isValid() method will slow down the system. 
	A complete overhaul is needed to fix isValid() method in order to increase performance and fix the small bug that system does not give any error for completely empty boards.
