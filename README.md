# Stock-Ticker
Sends a request for data of a stock to Yahoo Finance using a url.


/**
 * @authors Maxim Zibitsker, and Matthew Straughn
 * This program requests data from Yahoo Finance, which utilizes a URL 
 * connection in order to display information about a certain stock 
 * in both a GUI ticker, and a JTable.  T
 */
import java.awt.*;
import java.awt.event.*;
import java.net.URL;
import java.net.URLConnection;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.Scanner;
import java.util.TreeMap;
import java.util.TreeSet;

import javax.swing.*;
import javax.swing.table.DefaultTableModel;




public class finalP extends JPanel {


	/** Open a window on the screen showing a panel of type finalP
	 *  and its menu bar.  The window title is "FinalPanel" */
	public static void main (String[] args) {
		finalP panel = new finalP();
		JFrame window = new JFrame("FinaPanel");
		window.setContentPane(panel);
		window.setJMenuBar( panel.getMenuBar() );
		window.pack(); // Resize window based on content's preferred size
		window.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		window.setResizable(false); // User can't change window size.
		window.setVisible(true);  // Open the window on the screen.
	}// end of main
	
	//___________________INSTANCE VARIABLES____________________________________
	// Boolean used to test whether there is a stock with the associated symbol
	private Boolean isAStock; 
	// TreeMap used to store instances of StockItem with its associated data
	private TreeMap<String ,StockItem> items;
	// JPanel used to display instances of StockItem in a Ticker GUI
	private JPanel stockPanel;
	// tablePanel used to display instances of StockItem in a JTable
	private tablePanel tablePanel;
	// String array used to hold the ticker for each stock item
	String[] line; 
	// Integer used to index the array of strings 
	int index;
	// String that holds the scrolling stock info being printed in GUI ticker 
	String current;
	/*
	private final static String[] fileCommands = {
			"New", "Open...", "Save...", "Quit" 	
	};
	*/
	// String array that contains the names of the columns in the tablePanel
	private final static String[] columnNames = {
			"Symbol", "Name", "Change RT", "Change Per. RT", 
			"Day VC RT", "Ask", "Bid"
	};
	// Timer used to send a new URL connection every fifteen minutes
	private Timer updateF;
	// TreeSet used to store the stock symbols entered by the user
	private TreeSet <String> stockSymbols;
	
	/**
	 * Constructor 
	 * Displays the GUI ticker, and JTable
	 */
	public finalP() {
		this.setBackground(Color.WHITE);
		items = new TreeMap<String, StockItem>();
		this.setPreferredSize( new Dimension(800, 600) );
		this.setLayout(new BorderLayout() );
		stockPanel = new stockPanel();
		tablePanel = new tablePanel();
		this.add(stockPanel, BorderLayout.NORTH);
		this.add(tablePanel, BorderLayout.CENTER);
		isAStock =false;
	}// end of constructor

	//____________________ Stock Item Definition _________________________________-
	private class StockItem {
		String name; 	// name of company ticker
		String symbol; 	// symbol of the company
		String changeRT;		// change in real time
		String percentRT;		// percent change in real time
		String dayVC; //Day's value change
		String ask;	// current ask price
		String bid;	// current bid price
		String oldAsk;	// old ask price
		String oldBid;	// old bid price
		double diff;	// price fluctuation
		int row; //the row number in the table
		private StockItem(String symb,String n, String c6, String k2 , String w4, String askN, String bidN){
			// initialization
			name = n;
			symbol = symb;
			changeRT = c6;
			percentRT = k2;
			dayVC = w4;
			ask = askN;
			bid = bidN;
			oldAsk = "0";
			oldBid = "0";
			diff = 0;	// there should be no difference in price when created.
		}// end of constructor
		
		/**
		 * Returns the name of the stock item.
		 * @return name  the company name of the stock item.
		 */
		public String getName(){
			return name;
		}// end of getName()


		/**
		 * Returns the ticker symbol of the stock item.
		 * @return symbol the ticker symbol of the company.
		 */
		public String getSymbol(){
			return symbol;
		}// end of getSymbol()


		/**
		 * Returns the change, positive or negative, in the price of the stock item.
		 * @return c6  the positive or negative change on the price of stock item.
		 */
		public String getChange(){
			return changeRT;
		}// end of getChange()


		/**
		 * Returns the change in percent of the stock quote.
		 * @return k2  the change of the stock price in percent
		 */
		public String getPerChange(){
			return percentRT;
		}// end of getPerChange()

		/**
		 * Returns the change in percent of the stock quote.
		 * @return k2  the change of the stock price in percent
		 */
		public String getDayVCChange(){
			return dayVC;
		}// end of getDayVCChange()

		/**
		 * Returns the ask price.
		 * @return the ask price
		 */
		public String getAsk(){
			return ask;
		}// end of getAsk()

		/**
		 * Returns the bid price.
		 * @return the bid price
		 */
		public String getBid(){
			return bid;
		}// end of getBid()

		/**
		 * Returns the difference (+/-) in the stock price.
		 * @return the difference in the stock price
		 */
		public String getDiff(){
			return diff + "";
		}//end of getDiff()

		/**
		 * Returns the row number in the table where the stock information is located.
		 * @return the row number of a stock item in a table.
		 */
		public int getRow(){
			return row;
		}// end of getRow()
		
		/**
		 * Updates the ask and bid instance variable and reports the price 
		 * fluctuation in the stock.
		 * @param newAsk  the new ask price
		 * @param newBid  the new bid price
		 * @return diff The difference in the price fluctuation 
		 */
		public Double updatePrice(String newAsk, String newBid){
			// saves old data
			try{
				oldAsk = ask;
				oldBid = bid;

				//updates data
				ask = newAsk;
				bid = newBid;

				// compare the difference and report it
				Double convertAsk = Double.parseDouble(ask);
				Double convertBid = Double.parseDouble(bid);
				diff = convertAsk - convertBid;
				return diff ;
			}
			catch(Exception e){
				//System.out.println("Sorry there was no data available.");
				return 0.0;
			}
		}// end of updatePrice()
		//________________________________________________________
		/**
		 * Sets the name name of stock item.
		 * @param n holds the name of the company.
		 */
		public void setName(String n){
			name = n;
		}// end of getName()


		/**
		 * Sets the ticker value of the company.
		 * @param symb holds the ticker of the company.
		 */
		public void setSymbol(String symb){
			symbol = symb;
		}// end of getSymbol()	


		/**
		 * Sets the positive or negative change in price.
		 * @param num holds the change in stock price.
		 */
		public void setChange(String c6){
			changeRT = c6;
		}// end of getChange()


		/**
		 * Sets the change in percent of the stock quote price
		 * @param perChange the change in percent of the stock quote price
		 */
		public void setPerChange(String k2){
			percentRT = k2;
		}// end of getSymbol()		

		/**
		 * Sets the change in percent of the stock quote price
		 * @param perChange the change in percent of the stock quote price
		 */
		public void setDayVCChange(String w4){
			dayVC = w4;
		}// end of getSymbol()	
		/**

		 * Updates the current ask price of the stock.
		 * @param num the current ask price
		 */
		public void setAsk(String num){
			ask = num;
		}// end of setAsk()

		/**
		 * Updates the current bid price of the stock.
		 * @param num the current bid price
		 */
		public void setBid(String num){
			bid = num;
		}// end of setBid()

		/**
		 * Sets the row number of where the StockItem can be found in the table
		 * @param num the number of the row in the table where that StockItem is located
		 */
		public void setRow(int num){
			row = num;
		}// end of setRow()
		
	}// end of StockItem()

	//******************* Menu Handler **************************************
	public JMenuBar getMenuBar(){
		JMenuBar menubar = new JMenuBar();
		MenuHandler listener = new MenuHandler();
		JMenu stockMenu = new JMenu("Stock");
		JMenuItem addStock = new JMenuItem("Add Stock Item...");
		addStock.addActionListener(listener);
		stockMenu.add(addStock);
		stockMenu.addSeparator();
		menubar.add(stockMenu);
		return menubar;
	}// end of getMenuBar()

	/**
	 * MenuHandler used to let the user enter text, which will then create a
	 * URL connection to retrieve data from Yahoo Finance.
	 */
	private class MenuHandler implements ActionListener {

		public void actionPerformed(ActionEvent evt) {
			String command = evt.getActionCommand();

			if(command.equals("Add Stock Item...")){
				//debugging
				//stockItem stock1 = new stockItem("Apple", "APPL", "0", "0");
				String symbText = JOptionPane.showInputDialog(finalP.this, 
						"Enter the text for the stock symbol.");
				if (symbText == null || symbText.trim().length() == 0)
					// Don't add an item if user canceled or did not enter any text!
					return; 
				symbText = symbText.toUpperCase();
				try {

					URL url = new URL("http://finance.yahoo.com/d/quotes.csv?s=" + 
					symbText + "&f=sc6k2w4abn");
					URLConnection conn = url.openConnection();
					Scanner in = new Scanner(conn.getInputStream());
					String line = in.nextLine();
					String[] it = line.split(",", 7);
					String symb = it[0];
					symb = symb.substring(1, symb.length()-1);
					String c6 = it[1];
					String k2 = it[2];
					String w4 = it[3];
					String ask = it[4];
					String bid = it[5];
					String n = it[6];
					n = n.substring(1, n.length()-1);
					StockItem item = new StockItem(symb, n, c6, k2, w4, ask, bid );
					//Test if a stock exists
					if(n.equals("/")  ){
						isAStock = false;
						System.out.println("Stock does not exist");
					}//end if
					else{
						isAStock = true;

					}//end else
					if(isAStock == true){
						items.put(item.getSymbol(), item );
						item.updatePrice(ask, bid);
						tablePanel.update();
					}//end if
					
					//Debugging
					//System.out.println(item.getName());
					//System.out.println(line);
					in.close();
					repaint();
				}
				catch (Exception e) {
					System.out.println("Outlook is clouded at this time, try again later.");

					e.printStackTrace();
				}//end catch

			}// end of if
		}// end of actionPerformed()
	}// end of MenuHandler 

	/**
	 * Class to create a panel, which includes a JTable added to a scrollpane.
	 */
	public class tablePanel extends JPanel {
		private JTable table;
		public tablePanel(){
			this.setBackground(Color.WHITE);
			this.setLayout(new BorderLayout());
			DefaultTableModel model = new DefaultTableModel( columnNames, 100){
				public boolean isCellEditable(int row, int column ){
					return false;
				}
			};
			table = new JTable(model);
			table.setShowGrid(true);
			//Debugging
			//System.out.println("Added table with rows " + table.getRowCount());

			this.add(new JScrollPane(table), BorderLayout.CENTER);
			updateF = new Timer(900000, new ActionListener(){
				public void actionPerformed(ActionEvent e) {
					//Call update method 
					updateD();
					//debugging
					//System.out.println("UpdateD method works" );
				}
			});
			//Start thread
			updateF.start();
		}// end constructor

		/**
		 * Updates the table values with the information of each StockItem
		 */
		public synchronized void update(){
			
			int r = -1;//Variable for row
			int count = 0;//Variable for count
			line = new String[items.size()];//

			stockSymbols = new TreeSet<String>();

			for( String key: items.keySet() ){
				StockItem data = items.get(key);

				r++;
				//Create treeset to store all stock symbols
				//add stock symbols to treeset
				stockSymbols.add(data.getSymbol());
				
				
				//Test if increase or decrease
				if(data.updatePrice(data.ask, data.bid) > 0){
					//Display a plus symbol before the number
					line[count]=(data.name +" "  + data.symbol + " + "+ String.format("%1.2f",Double.parseDouble(data.getDiff())) );
					
				}else{
					//Display a minus symbol before the number
					line[count]=(data.name +" "  + data.symbol + " - "+ String.format("%1.2f",Double.parseDouble(data.getDiff())) );
				}
				
				// adds stock data into the array of strings for each respective StockItem
				//line[count]=(data.name +" "  + data.symbol + " "+ String.format("%1.2f",Double.parseDouble(data.getDiff())) ); 
				count++;


				table.setValueAt(data.getSymbol(), r, 0);
				table.setValueAt(data.getName(), r, 1);
				table.setValueAt(data.getChange(), r, 2);
				table.setValueAt(data.getPerChange() , r, 3);
				table.setValueAt(data.getDayVCChange() , r, 4);
				table.setValueAt(data.getAsk(), r, 5);
				table.setValueAt(data.getBid(), r, 6);
				data.setRow(r);  // sets the row location for each stock item

				System.out.println("Here you go" + stockSymbols);

			}// end of for each loop


		}// end of update()

		/**
		 * Updates the table with new data after sending a URL connection.
		 */
		public void updateD(){

			if(stockSymbols == null || stockSymbols.isEmpty()){
				return;
			}
			Iterator <String> itS = stockSymbols.iterator();

			String URLString = "http://finance.yahoo.com/d/quotes.csv?s=";
			boolean first = true;
			while(itS.hasNext()) {
				if (!first) {
					URLString += "+";
				}
				URLString += itS.next();
				first = false;
			}//end of while
			URLString += "&f=sc6k2w4abn";
			System.out.println("Sending " + URLString);


			try {


				URL url = new URL(URLString);
				URLConnection conn = url.openConnection();
				Scanner in = new Scanner(conn.getInputStream());
				while (in.hasNextLine()) {
					String line = in.nextLine();
					String[] it = line.split(",", 7);


					String symb = it[0];
					symb = symb.substring(1, symb.length()-1);



					String c6 = it[1];
					String k2 = it[2];
					String w4 = it[3];

					String ask = it[4];
					String bid = it[5];
					//System.out.println("here");
					String n = it[6];

					n = n.substring(1, n.length()-1);
					//System.out.println("name: " + n);
					StockItem item = new StockItem(symb, n, c6, k2, w4, ask, bid );

					items.put(item.getSymbol(), item);
				}//end of while
				
			} catch(Exception E){

			}// end of catch
		}// end of updateD


	}// end of tablePanel





	/**
	 * Class to create the GUI ticker
	 */
	public class stockPanel extends JPanel implements ActionListener {
		//Local Variables
		int x = -1000;// initial x-position
		int y = 100;// y-position
		int counter = 0;// counter variable
		
		/**
		 * Constructor for stockPanel, which creates a timer and starts it.
		 */
		public stockPanel(){
			this.setBackground(Color.BLACK);
			this.setPreferredSize(new Dimension(1000,200));
			Timer frameTimer = new Timer(100, this);
			frameTimer.start();

		}//end of stockPanel

		/**
		 * Paints the String that is being drawn on the stockPanel
		 */
		public void paintComponent(Graphics painter){
			super.paintComponent(painter);
			//Display grid
			fractalCloud(painter,0, 0, 1000, 200, 110, 130, 190,170, 400);
			//Test to see if the TreeMap is empty, if so return
			if(items.isEmpty()){
				return;
			}//end of if
			
			//Create a new font, set the color, and font of the string 
			//that will be displayed to the user.
			Font font = new Font("Garramond", Font.BOLD, 46);
			painter.setColor(Color.YELLOW);
			painter.setFont(font);
			//Assign a compartment in the array with the stock item
			current = line[index];
			painter.drawString(current, x , y);
		}//end painter
		
		/**
		 * Allows the rest of the drawn string's position when it leaves the screen
		 * @param e, ActionEvent used to animate the string being displayed to the user.
		 */
		public void actionPerformed(ActionEvent e) {
			x +=10;
			//Test if the string is drawn off the page
			if(x>this.getWidth()){
				x = -1000;
				//Test if the index is one less than the size of the TreeMap
				if(index < items.size() -1){
					index++;
				}//end if
				else{
					index =0;
				}//end else
			}//end if
			repaint();
		}// end actionPerformed(ActionEvent e)
	}// end of stockPanel

	/**
	 * A recursive method that creates the grid shown in the in the stockPanel. 
	 */
	private void fractalCloud(Graphics painter,double x1, double y1,double w, double h, double c0,
			double c2, double c4, double c8, double dis){
		//local variable that oscillates between a positive and negative random distance
		double d = ((Math.random() * dis));
		//test 50% probability of being either positive or negative
		if(Math.random()>.5){
			d=-1.0*d;
		}
		//tests if width is less than 10
		if(w <10){
			//local variable to store the value of the red and green component
			// of the objects color
			int r=((int)(c0+c2+c4+c8)/4)%255;//cast doubles into integers
			r=Math.max(r,1);//returns the greater of the two numbers
			painter.setColor(Color.GRAY);//set new color
			painter.fillRect((int)x1,(int)y1,(int)w,(int)h);//draw a filled in
			return;
		} else{
			// draws the top half of the fractal cloud 
			fractalCloud( painter,x1, y1, w/2, h/2, c0, (c0+c2)/2, (c0+c4)/2,
					+(c0+c2+c4+c8)/4 +d, dis/2 );
			fractalCloud( painter,x1 +w/2, y1, w/2, h/2, (c0+c2)/2, c2,
					+ (c0+c2+c4+c8)/4 +d, (c2+c8)/2, dis/2);

			// draws the bottom half of the fractal cloud
			fractalCloud( painter,x1, y1 +h/2, w/2, h/2, (c0+c4)/2, (c0+c2+c4+c8)/4 +d,
					+c4, (c4+c8)/2, dis/2 );
			fractalCloud( painter, x1+ w/2, y1 +h/2, w/2, h/2, (c0+c2+c4+c8)/4 +d,
					+(c2+c8)/2, (c4+c8)/2, c8, dis/2);
		}//end else
	}//end fractalCloud()
}// end of final()





