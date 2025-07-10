# Browser-history-manager
import javax.swing.*;
import java.awt.*;
import java.util.*;

public class BrowserHistoryManager extends JFrame {
    JTextField urlInput;
    JTextArea historyDisplay, backStackDisplay, forwardStackDisplay, linkedListDisplay;
    JButton visitBtn, backBtn, forwardBtn, showHistoryBtn, clearHistoryBtn;

    Node head = null, tail = null, current = null;
    Stack<String> backStack = new Stack<>();
    Stack<String> forwardStack = new Stack<>();

    class Node {
        String url;
        Node prev, next;

        Node(String url) {
            this.url = url;
        }
    }

    public BrowserHistoryManager() {
        setTitle("Browser History Manager");
        setSize(800, 600);
        setLayout(new BorderLayout());
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        // Top Panel (URL input + Visit)
        JPanel topPanel = new JPanel(new BorderLayout());
        urlInput = new JTextField();
        visitBtn = new JButton("Visit");
        topPanel.add(urlInput, BorderLayout.CENTER);
        topPanel.add(visitBtn, BorderLayout.EAST);

        // Control Buttons
        JPanel controlPanel = new JPanel();
        backBtn = new JButton("Back");
        forwardBtn = new JButton("Forward");
        showHistoryBtn = new JButton("Show History");
        clearHistoryBtn = new JButton("Clear History"); // New button
        controlPanel.add(backBtn);
        controlPanel.add(forwardBtn);
        controlPanel.add(showHistoryBtn);
        controlPanel.add(clearHistoryBtn); // Add to control panel

        // Displays
        historyDisplay = new JTextArea(10, 50);
        historyDisplay.setEditable(false);
        JScrollPane historyScroll = new JScrollPane(historyDisplay);

        backStackDisplay = new JTextArea(10, 15);
        forwardStackDisplay = new JTextArea(10, 15);
        backStackDisplay.setEditable(false);
        forwardStackDisplay.setEditable(false);

        JPanel stackPanel = new JPanel(new GridLayout(1, 2));
        stackPanel.add(new JScrollPane(backStackDisplay));
        stackPanel.add(new JScrollPane(forwardStackDisplay));

        JPanel labelPanel = new JPanel(new GridLayout(1, 2));
        labelPanel.add(new JLabel("Back Stack", SwingConstants.CENTER));
        labelPanel.add(new JLabel("Forward Stack", SwingConstants.CENTER));

        linkedListDisplay = new JTextArea(3, 50);
        linkedListDisplay.setEditable(false);

        // Layout
        add(topPanel, BorderLayout.NORTH);
        add(controlPanel, BorderLayout.CENTER);
        add(historyScroll, BorderLayout.WEST);
        add(stackPanel, BorderLayout.EAST);
        add(labelPanel, BorderLayout.SOUTH);
        add(linkedListDisplay, BorderLayout.SOUTH);

        // Action Listeners
        visitBtn.addActionListener(e -> visitPage());
        backBtn.addActionListener(e -> goBack());
        forwardBtn.addActionListener(e -> goForward());
        showHistoryBtn.addActionListener(e -> showHistory());
        clearHistoryBtn.addActionListener(e -> clearHistory()); // NEW

        // Seed data
        addPage("https://example.com");
        addPage("https://google.com");
        addPage("https://github.com");

        updateUI();
    }

    void visitPage() {
        String url = urlInput.getText().trim();
        if (url.isEmpty()) return;

        if (current != null) {
            backStack.push(current.url);
        }

        forwardStack.clear();
        addPage(url);
        urlInput.setText("");
        updateUI();
    }

    void addPage(String url) {
        Node newNode = new Node(url);
        if (head == null) {
            head = tail = newNode;
        } else {
            tail.next = newNode;
            newNode.prev = tail;
            tail = newNode;
        }
        current = newNode;
    }

    void goBack() {
        if (current != null && current.prev != null) {
            forwardStack.push(current.url);
            current = current.prev;
            backStack.pop();
        } else if (!backStack.isEmpty()) {
            forwardStack.push(current.url);
            current = findNode(backStack.pop());
        } else {
            JOptionPane.showMessageDialog(this, "No more back history");
        }
        updateUI();
    }

    void goForward() {
        if (!forwardStack.isEmpty()) {
            backStack.push(current.url);
            current = findNode(forwardStack.pop());
        } else {
            JOptionPane.showMessageDialog(this, "No more forward history");
        }
        updateUI();
    }

    void showHistory() {
        java.util.List<String> urls = getAllHistory();
        historyDisplay.setText("History:\n");
        for (String url : urls) {
            historyDisplay.append(url + (current != null && url.equals(current.url) ? " (current)\n" : "\n"));
        }
    }

    java.util.List<String> getAllHistory() {
        java.util.List<String> list = new ArrayList<>();
        Node temp = head;
        while (temp != null) {
            list.add(temp.url);
            temp = temp.next;
        }
        return list;
    }

    Node findNode(String url) {
        Node temp = head;
        while (temp != null) {
            if (temp.url.equals(url)) return temp;
            temp = temp.next;
        }
        return null;
    }

    void clearHistory() {
        int confirm = JOptionPane.showConfirmDialog(this, "Are you sure you want to clear all history?", "Confirm", JOptionPane.YES_NO_OPTION);
        if (confirm == JOptionPane.YES_OPTION) {
            head = null;
            tail = null;
            current = null;
            backStack.clear();
            forwardStack.clear();
            historyDisplay.setText("");
            JOptionPane.showMessageDialog(this, "History cleared.");
            updateUI();
        }
    }

    void updateUI() {
        // Update stacks
        backStackDisplay.setText("Back Stack:\n");
        for (String url : backStack) {
            backStackDisplay.append(url + "\n");
        }

        forwardStackDisplay.setText("Forward Stack:\n");
        for (String url : forwardStack) {
            forwardStackDisplay.append(url + "\n");
        }

        // Linked list visual
        StringBuilder listBuilder = new StringBuilder("Linked List:\n");
        Node temp = head;
        while (temp != null) {
            listBuilder.append(temp == current ? "[" + temp.url + "]" : temp.url);
            if (temp.next != null) listBuilder.append(" <-> ");
            temp = temp.next;
        }
        linkedListDisplay.setText(listBuilder.toString());
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            new BrowserHistoryManager().setVisible(true);
        });
    }
}
