package javaconnectivity;
package javaconnectivity;

import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.sql.*;
import java.text.SimpleDateFormat;
import java.util.Date;

public class AuctionManagementSystem {
    private static final String DB_URL = "jdbc:mysql://localhost:3306/auction23system";
    private static final String DB_USER = "root";
    private static final String DB_PASSWORD = "sasmitha@2005";

    private Connection connection;

    public AuctionManagementSystem() {
        try {
            connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
            System.out.println("Connected to the database.");
            showMainMenu();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(AuctionManagementSystem::new);
    }

    private void showMainMenu() {
        JFrame frame = new JFrame("Auction Management System");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 300);

        JPanel panel = new JPanel();
        panel.setLayout(new GridLayout(5, 1, 10, 10));

        JButton registerButton = new JButton("Register User");
        JButton loginButton = new JButton("Login");
        JButton createAuctionButton = new JButton("Create Auction");
        JButton viewAuctionsButton = new JButton("View Auctions");
        JButton placeBidButton = new JButton("Place Bid");
        JButton viewResultsButton = new JButton("View Auction Results");
       
        

        registerButton.addActionListener(e -> showRegisterForm(frame));
        loginButton.addActionListener(e -> showLoginForm(frame));
        createAuctionButton.addActionListener(e -> showCreateAuctionForm(frame));
        viewAuctionsButton.addActionListener(e -> showAuctionList(frame));
        placeBidButton.addActionListener(e -> showPlaceBidForm(frame));
        viewResultsButton.addActionListener(e -> showAuctionResults(frame));

        panel.add(registerButton);
        panel.add(loginButton);
        panel.add(createAuctionButton);
        panel.add(viewAuctionsButton);
        panel.add(placeBidButton);
        panel.add(viewResultsButton);

        frame.add(panel);
        frame.setVisible(true);
    }

    private void showRegisterForm(JFrame parentFrame) {
        JFrame frame = new JFrame("Register User");
        frame.setSize(300, 250);

        JPanel panel = new JPanel(new GridLayout(4, 2, 10, 10));
        JLabel usernameLabel = new JLabel("Username:");
        JTextField usernameField = new JTextField();
        JLabel passwordLabel = new JLabel("Password:");
        JPasswordField passwordField = new JPasswordField();
        JLabel emailLabel = new JLabel("Email:");
        JTextField emailField = new JTextField();
        JButton registerButton = new JButton("Register");

        registerButton.addActionListener(e -> {
            String username = usernameField.getText();
            String password = new String(passwordField.getPassword());
            String email = emailField.getText();
            registerUser(username, password, email);
            frame.dispose();
        });

        panel.add(usernameLabel);
        panel.add(usernameField);
        panel.add(passwordLabel);
        panel.add(passwordField);
        panel.add(emailLabel);
        panel.add(emailField);
        panel.add(new JLabel()); // Empty space
        panel.add(registerButton);

        frame.add(panel);
        frame.setVisible(true);
    }

    private void registerUser(String username, String password, String email) {
        try {
            String sql = "INSERT INTO users (username, password, email) VALUES (?, ?, ?)";
            PreparedStatement statement = connection.prepareStatement(sql);
            statement.setString(1, username);
            statement.setString(2, password);
            statement.setString(3, email);
            statement.executeUpdate();
            JOptionPane.showMessageDialog(null, "User registered successfully!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void showLoginForm(JFrame parentFrame) {
        JFrame frame = new JFrame("Login");
        frame.setSize(300, 200);

        JPanel panel = new JPanel(new GridLayout(3, 2, 10, 10));
        JLabel usernameLabel = new JLabel("Username:");
        JTextField usernameField = new JTextField();
        JLabel passwordLabel = new JLabel("Password:");
        JPasswordField passwordField = new JPasswordField();
        JButton loginButton = new JButton("Login");

        loginButton.addActionListener(e -> {
            String username = usernameField.getText();
            String password = new String(passwordField.getPassword());
            if (loginUser(username, password)) {
                JOptionPane.showMessageDialog(null, "Login successful!");
                frame.dispose();
            } else {
                JOptionPane.showMessageDialog(null, "Invalid credentials.");
            }
        });

        panel.add(usernameLabel);
        panel.add(usernameField);
        panel.add(passwordLabel);
        panel.add(passwordField);
        panel.add(new JLabel()); // Empty space
        panel.add(loginButton);

        frame.add(panel);
        frame.setVisible(true);
    }

    private boolean loginUser(String username, String password) {
        try {
            String sql = "SELECT * FROM users WHERE username = ? AND password = ?";
            PreparedStatement statement = connection.prepareStatement(sql);
            statement.setString(1, username);
            statement.setString(2, password);
            return statement.executeQuery().next();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return false;
    }

    private void showCreateAuctionForm(JFrame parentFrame) {
        JFrame frame = new JFrame("Create Auction");
        frame.setSize(300, 250);

        JPanel panel = new JPanel(new GridLayout(4, 2, 10, 10));
        JLabel itemNameLabel = new JLabel("Item Name:");
        JTextField itemNameField = new JTextField();
        JLabel startingPriceLabel = new JLabel("Starting Price:");
        JTextField startingPriceField = new JTextField();
        JLabel endTimeLabel = new JLabel("End Time (yyyy-MM-dd HH:mm:ss):");
        JTextField endTimeField = new JTextField();
        JButton createButton = new JButton("Create");

        createButton.addActionListener(e -> {
            String itemName = itemNameField.getText();
            double startingPrice = Double.parseDouble(startingPriceField.getText());
            String endTimeStr = endTimeField.getText();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            Date endTime = null;
            try {
                endTime = sdf.parse(endTimeStr);
            } catch (Exception ex) {
                ex.printStackTrace();
            }
            createAuction(itemName, startingPrice, new Timestamp(endTime.getTime()));
            frame.dispose();
        });

        panel.add(itemNameLabel);
        panel.add(itemNameField);
        panel.add(startingPriceLabel);
        panel.add(startingPriceField);
        panel.add(endTimeLabel);
        panel.add(endTimeField);
        panel.add(new JLabel()); // Empty space
        panel.add(createButton);

        frame.add(panel);
        frame.setVisible(true);
    }

    private void createAuction(String itemName, double startingPrice, Timestamp endTime) {
        try {
            String sql = "INSERT INTO auctions (item_name, starting_price, end_time) VALUES (?, ?, ?)";
            PreparedStatement statement = connection.prepareStatement(sql);
            statement.setString(1, itemName);
            statement.setDouble(2, startingPrice);
            statement.setTimestamp(3, endTime);
            statement.executeUpdate();
            JOptionPane.showMessageDialog(null, "Auction created successfully!");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void showPlaceBidForm(JFrame parentFrame) {
        JFrame frame = new JFrame("Place Bid");
        frame.setSize(300, 250);

        JPanel panel = new JPanel(new GridLayout(4, 2, 10, 10));
        JLabel auctionIdLabel = new JLabel("Auction ID:");
        JTextField auctionIdField = new JTextField();
        JLabel bidAmountLabel = new JLabel("Bid Amount:");
        JTextField bidAmountField = new JTextField();
        JLabel usernameLabel = new JLabel("Your Username:");
        JTextField usernameField = new JTextField();
        JButton bidButton = new JButton("Place Bid");

        bidButton.addActionListener(e -> {
            int auctionId = Integer.parseInt(auctionIdField.getText());
            double bidAmount = Double.parseDouble(bidAmountField.getText());
            String username = usernameField.getText();
            placeBid(username, auctionId, bidAmount);
            frame.dispose();
        });

        panel.add(auctionIdLabel);
        panel.add(auctionIdField);
        panel.add(bidAmountLabel);
        panel.add(bidAmountField);
        panel.add(usernameLabel);
        panel.add(usernameField);
        panel.add(new JLabel()); // Empty space
        panel.add(bidButton);

        frame.add(panel);
        frame.setVisible(true);
    }
    private void showAuctionList(JFrame parentFrame) {
        JFrame frame = new JFrame("List of Auctions");
        frame.setSize(600, 400);

        String[] columnNames = {"Auction ID", "Item Name", "Starting Price", "End Time"};
        DefaultTableModel tableModel = new DefaultTableModel(columnNames, 0);

        try {
            // Query to fetch auction details
            String sql = "SELECT auction_id, item_name, starting_price, end_time FROM auctions";
            PreparedStatement statement = connection.prepareStatement(sql);
            ResultSet resultSet = statement.executeQuery();

            while (resultSet.next()) {
                int auctionId = resultSet.getInt("auction_id");
                String itemName = resultSet.getString("item_name");
                double startingPrice = resultSet.getDouble("starting_price");
                Timestamp endTime = resultSet.getTimestamp("end_time");

                // Adding rows to the table model
                tableModel.addRow(new Object[]{auctionId, itemName, startingPrice, endTime});
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }

        // Create JTable to display auction list
        JTable table = new JTable(tableModel);
        JScrollPane scrollPane = new JScrollPane(table);

        frame.add(scrollPane);
        frame.setVisible(true);
    }


    private void placeBid(String username, int auctionId, double bidAmount) {
        try {
            // Check if the bid is higher than the current highest bid
            String checkBidSql = "SELECT highest_bid FROM auctions WHERE auction_id = ?";
            PreparedStatement checkBidStatement = connection.prepareStatement(checkBidSql);
            checkBidStatement.setInt(1, auctionId);
            ResultSet resultSet = checkBidStatement.executeQuery();
            if (resultSet.next()) {
                double highestBid = resultSet.getDouble("highest_bid");
                if (bidAmount <= highestBid) {
                    JOptionPane.showMessageDialog(null, "Bid must be higher than the current highest bid.");
                    return;
                }
            }

            // Place the bid
            String getUserIdSql = "SELECT user_id FROM users WHERE username = ?";
            PreparedStatement getUserIdStatement = connection.prepareStatement(getUserIdSql);
            getUserIdStatement.setString(1, username);
            ResultSet userResult = getUserIdStatement.executeQuery();
            if (userResult.next()) {
                int userId = userResult.getInt("user_id");

                String insertBidSql = "INSERT INTO bids (auction_id, user_id, bid_amount) VALUES (?, ?, ?)";
                PreparedStatement insertBidStatement = connection.prepareStatement(insertBidSql);
                insertBidStatement.setInt(1, auctionId);
                insertBidStatement.setInt(2, userId);
                insertBidStatement.setDouble(3, bidAmount);
                insertBidStatement.executeUpdate();

                // Update the highest bid for the auction
                String updateAuctionSql = "UPDATE auctions SET highest_bid = ?, winner_id = ? WHERE auction_id = ?";
                PreparedStatement updateAuctionStatement = connection.prepareStatement(updateAuctionSql);
                updateAuctionStatement.setDouble(1, bidAmount);
                updateAuctionStatement.setInt(2, userId);
                updateAuctionStatement.setInt(3, auctionId);
                updateAuctionStatement.executeUpdate();

                JOptionPane.showMessageDialog(null, "Bid placed successfully!");
            } else {
                JOptionPane.showMessageDialog(null, "User not found.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void showAuctionResults(JFrame parentFrame) {
        JFrame frame = new JFrame("Auction Results");
        frame.setSize(600, 400);

        JPanel panel = new JPanel(new BorderLayout());
        String[] columnNames = {"Auction ID", "Item Name", "Starting Price", "Highest Bid", "Winner"};
        DefaultTableModel model = new DefaultTableModel(columnNames, 0);
        JTable table = new JTable(model);
        JScrollPane scrollPane = new JScrollPane(table);
        panel.add(scrollPane, BorderLayout.CENTER);

        // Fetch auction data and display it
        try {
            String sql = "SELECT a.auction_id, a.item_name, a.starting_price, a.highest_bid, u.username AS winner " +
                         "FROM auctions a LEFT JOIN users u ON a.winner_id = u.user_id";
            PreparedStatement statement = connection.prepareStatement(sql);
            ResultSet resultSet = statement.executeQuery();

            while (resultSet.next()) {
                int auctionId = resultSet.getInt("auction_id");
                String itemName = resultSet.getString("item_name");
                double startingPrice = resultSet.getDouble("starting_price");
                double highestBid = resultSet.getDouble("highest_bid");
                String winner = resultSet.getString("winner");

                model.addRow(new Object[]{auctionId, itemName, startingPrice, highestBid, winner});
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }

        frame.add(panel);
        frame.setVisible(true);
    }
}




MYSQL SCHEMA
CREATE DATABASE auction23system;

USE auction23system;

-- Users Table (updated with email)
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);

-- Auctions Table (with start price, end time)
CREATE TABLE auctions (
    auction_id INT AUTO_INCREMENT PRIMARY KEY,
    item_name VARCHAR(100) NOT NULL,
    starting_price DOUBLE NOT NULL,
    end_time TIMESTAMP NOT NULL,
    highest_bid DOUBLE DEFAULT 0,
    winner_id INT,
    FOREIGN KEY (winner_id) REFERENCES users(user_id)
);

-- Bids Table (recording bid details with user reference)
CREATE TABLE bids (
    bid_id INT AUTO_INCREMENT PRIMARY KEY,
    auction_id INT,
    user_id INT,
    bid_amount DOUBLE,
    bid_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (auction_id) REFERENCES auctions(auction_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
