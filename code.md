# Final-Expense-Management-Application
import java.util.ArrayList;
import java.util.*;
import java.io.File;
import java.io.FileWriter;
import java.io.PrintWriter;
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.HashMap;
import java.time.LocalDate;

public class FinancialExpense {

    // Name of the file used to save and load records.
    // Each line in the file is one record with this format:
    // date, Type, category, amount
    // Example line: 2025-09-28, Income, salary, 2500.50
    public static final String DATA_FILE = "data.txt";

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // In-memory list of records (each record is a single String).
        // We use ArrayList so we can add and remove easily.
        ArrayList<String> records = new ArrayList<String>();

        // Load saved records from the file into the ArrayList when program starts.
        loadFromFile(records);

        // Main menu loop
        while (true) {
            System.out.println("\n--- Simple Finance App ---");
            System.out.println("1 Add Income");
            System.out.println("2 Add Expense");
            System.out.println("3 View All Records");
            System.out.println("4 Update Record");
            System.out.println("5 Delete Record");
            System.out.println("6 View Totals (income/expense/balance)");
            System.out.println("7 View Category Summary");
            System.out.println("8 Exit");
            System.out.print("Choose (1-8): ");

            String line = sc.nextLine().trim();
            int choice;
            try {
                choice = Integer.parseInt(line);
            } catch (NumberFormatException e) {
                System.out.println("Enter a number between 1 and 8.");
                continue;
            }

            if (choice == 1) {
                addRecord(records, sc, true);   // add income
            } else if (choice == 2) {
                addRecord(records, sc, false);  // add expense
            } else if (choice == 3) {
                viewRecords(records);
            } else if (choice == 4) {
                updateRecord(records, sc);
            } else if (choice == 5) {
                deleteRecord(records, sc);
            } else if (choice == 6) {
                viewTotals(records);
            } else if (choice == 7) {
                viewCategorySummary(records);
            } else if (choice == 8) {
                System.out.println("Exiting. All data saved to " + DATA_FILE);
                break;
            } else {
                System.out.println("Please choose a valid option (1-8).");
            }
        }

        sc.close();
    }

    // File load/save

    // Read data.txt line by line and add each non-empty line to records.
    public static void loadFromFile(ArrayList<String> records) {
        try {
            File f = new File(DATA_FILE);
            if (!f.exists()) {
                // Create the file so next runs find it
                f.createNewFile();
                return;
            }
            BufferedReader br = new BufferedReader(new FileReader(f));
            String line;
            while ((line = br.readLine()) != null) {
                if (!line.trim().isEmpty()) {
                    records.add(line);
                }
            }
            br.close();
        } catch (IOException e) {
            System.out.println("Could not load saved data: " + e.getMessage());
        }
    }

    // Overwrite the data file with the current content of records.
    // We call this after any change (add/update/delete) so file and memory match.
    public static void saveToFile(ArrayList<String> records) {
        try {
            PrintWriter pw = new PrintWriter(new FileWriter(DATA_FILE));
            for (int i = 0; i < records.size(); i++) {
                pw.println(records.get(i));
            }
            pw.close();
        } catch (IOException e) {
            System.out.println("Error saving data: " + e.getMessage());
        }
    }

    // Add a new income or expense. date is automatically added as today's date.
    public static void addRecord(ArrayList<String> records, Scanner sc, boolean isIncome) {
        System.out.print("Enter category (food, travel, salary, rent etc): ");
        String category = sc.nextLine().trim();
        if (category.isEmpty()) category = "other";

        System.out.print("Enter amount (use decimals allowed, e.g. 123.45): ");
        String amtLine = sc.nextLine().trim();
        double amount;
        try {
            amount = Double.parseDouble(amtLine);
            if (amount < 0) {
                System.out.println("Amount must be positive.");
                return;
            }
        } catch (NumberFormatException e) {
            System.out.println("Invalid amount. Use numbers like 250 or 99.50");
            return;
        }

        // Today's date (ISO format yyyy-MM-dd)
        String date = LocalDate.now().toString();

        // Build record string: date, Type, category, amount
        String type = isIncome ? "Income" : "Expense";
        String record = date + ", " + type + ", " + category + ", " + amount;

        // Add to ArrayList in memory
        records.add(record);

        // Save immediately so data persists
        saveToFile(records);
        System.out.println(type + " added.");
    }

    // Show all saved records with index numbers (so user can update/delete by number)
    public static void viewRecords(ArrayList<String> records) {
        System.out.println("\n--- All Records ---");
        if (records.size() == 0) {
            System.out.println("No records found.");
            return;
        }
        for (int i = 0; i < records.size(); i++) {
            System.out.println((i + 1) + ". " + records.get(i));
        }
    }

    // Update a record chosen by its number. We keep the original date.
    public static void updateRecord(ArrayList<String> records, Scanner sc) {
        if (records.size() == 0) {
            System.out.println("No records to update.");
            return;
        }
        viewRecords(records);
        System.out.print("Enter record number to update: ");
        String numLine = sc.nextLine().trim();
        int num;
        try {
            num = Integer.parseInt(numLine);
        } catch (NumberFormatException e) {
            System.out.println("Invalid number.");
            return;
        }
        if (num < 1 || num > records.size()) {
            System.out.println("Number out of range.");
            return;
        }

        String old = records.get(num - 1);
        // expected format: date, Type, category, amount
        String[] parts = old.split(", ");
        if (parts.length < 4) {
            System.out.println("Record format unexpected; cannot update.");
            return;
        }
        String date = parts[0]; // keep original date

        System.out.print("Enter new type (Income or Expense) [current: " + parts[1] + "]: ");
        String newType = sc.nextLine().trim();
        if (!(newType.equalsIgnoreCase("Income") || newType.equalsIgnoreCase("Expense"))) {
            System.out.println("Type must be Income or Expense.");
            return;
        }
        newType = newType.substring(0,1).toUpperCase() + newType.substring(1).toLowerCase();

        System.out.print("Enter new category [current: " + parts[2] + "]: ");
        String newCat = sc.nextLine().trim();
        if (newCat.isEmpty()) newCat = parts[2];

        System.out.print("Enter new amount [current: " + parts[3] + "]: ");
        String newAmtLine = sc.nextLine().trim();
        double newAmt;
        try {
            newAmt = Double.parseDouble(newAmtLine);
            if (newAmt < 0) {
                System.out.println("Amount must be positive.");
                return;
            }
        } catch (NumberFormatException e) {
            System.out.println("Invalid amount.");
            return;
        }

        String newRecord = date + ", " + newType + ", " + newCat + ", " + newAmt;
        records.set(num - 1, newRecord);
        saveToFile(records);
        System.out.println("Record updated.");
    }

    // Delete chosen record by number
    public static void deleteRecord(ArrayList<String> records, Scanner sc) {
        if (records.size() == 0) {
            System.out.println("No records to delete.");
            return;
        }
        viewRecords(records);
        System.out.print("Enter record number to delete: ");
        String numLine = sc.nextLine().trim();
        int num;
        try {
            num = Integer.parseInt(numLine);
        } catch (NumberFormatException e) {
            System.out.println("Invalid number.");
            return;
        }
        if (num < 1 || num > records.size()) {
            System.out.println("Number out of range.");
            return;
        }
        // Remove from list and save
        records.remove(num - 1);
        saveToFile(records);
        System.out.println("Record deleted.");
    }

    // Show total income, total expense, and remaining balance
    public static void viewTotals(ArrayList<String> records) {
        double totalIncome = 0.0;
        double totalExpense = 0.0;

        for (int i = 0; i < records.size(); i++) {
            String r = records.get(i);
            String[] parts = r.split(", ");
            if (parts.length < 4) continue;
            String type = parts[1];
            double amt;
            try {
                amt = Double.parseDouble(parts[3]);
            } catch (NumberFormatException e) {
                continue;
            }
            if (type.equals("Income")) {
                totalIncome += amt;
            } else {
                totalExpense += amt;
            }
        }

        System.out.println("\nTotal Income = " + String.format("%.2f", totalIncome));
        System.out.println("Total Expense = " + String.format("%.2f", totalExpense));
        System.out.println("Remaining Balance = " + String.format("%.2f", (totalIncome - totalExpense)));
    }

    // Show totals grouped by category separately for Income and Expense.
    public static void viewCategorySummary(ArrayList<String> records) {
        // Maps to store category -> total amount
        HashMap<String, Double> incomeMap = new HashMap<String, Double>();
        HashMap<String, Double> expenseMap = new HashMap<String, Double>();

        for (int i = 0; i < records.size(); i++) {
            String r = records.get(i);
            String[] parts = r.split(", ");
            if (parts.length < 4) continue;
            String type = parts[1];
            String category = parts[2];
            double amt;
            try {
                amt = Double.parseDouble(parts[3]);
            } catch (NumberFormatException e) {
                continue;
            }

            if (type.equals("Income")) {
                if (incomeMap.containsKey(category)) {
                    incomeMap.put(category, incomeMap.get(category) + amt);
                } else {
                    incomeMap.put(category, amt);
                }
            } else { // Expense
                if (expenseMap.containsKey(category)) {
                    expenseMap.put(category, expenseMap.get(category) + amt);
                } else {
                    expenseMap.put(category, amt);
                }
            }
        }

        System.out.println("\n--- Income by Category ---");
        Object[] iKeys = incomeMap.keySet().toArray();
        if (iKeys.length == 0) {
            System.out.println("No income records.");
        } else {
            for (int k = 0; k < iKeys.length; k++) {
                String cat = (String) iKeys[k];
                double total = incomeMap.get(cat);
                System.out.println(cat + " -> " + String.format("%.2f", total));
            }
        }

        System.out.println("\n---Expense by Category ---");
        Object[] eKeys = expenseMap.keySet().toArray();
        if (eKeys.length == 0) {
            System.out.println("No expense records.");
        } else {
            for (int k = 0; k < eKeys.length; k++) {
                String cat = (String) eKeys[k];
                double total = expenseMap.get(cat);
                System.out.println(cat + " -> " + String.format("%.2f", total));
            }
        }
    }
}
