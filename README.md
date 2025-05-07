# Lab6-OOP
Main:
package org.example;

import org.example.db.DatabaseException;
import org.example.db.DatabaseManager;
import org.example.model.SportSection;
import org.example.view.SportsSectionGUI;
import org.example.controller.SportsSectionController;
import javax.swing.*;

public class MainApp {
    public static void main(String[] args) {
        // Запуск в Event Dispatch Thread для Swing
        try {
            DatabaseManager.initialize();
        } catch (DatabaseException e) {
            throw new RuntimeException(e);
        }

        SwingUtilities.invokeLater(() -> {
            try {
                // Инициализация MVC-компонентов
                SportSection model = new SportSection();
                SportsSectionGUI view = new SportsSectionGUI();
                new SportsSectionController(model, view);

                // Настройка и отображение главного окна
                configureMainWindow(view);
            } catch (Exception e) {
                showFatalError(e);
            }
        });
    }

    private static void configureMainWindow(SportsSectionGUI view) {
        view.setLocationRelativeTo(null); // Центрирование окна
        view.setResizable(true);
        view.setDefaultCloseOperation(WindowConstants.DO_NOTHING_ON_CLOSE);

        // Обработчик закрытия окна
        view.addWindowListener(new java.awt.event.WindowAdapter() {
            @Override
            public void windowClosing(java.awt.event.WindowEvent e) {
                confirmExit(view);
            }
        });

        view.setVisible(true);
    }

    private static void confirmExit(JFrame parent) {
        int choice = JOptionPane.showConfirmDialog(
                parent,
                "Сохранить данные перед выходом?",
                "Подтверждение выхода",
                JOptionPane.YES_NO_CANCEL_OPTION
        );

        if (choice == JOptionPane.YES_OPTION) {
            // Контроллер сохранит данные через view
            parent.dispose();
            System.exit(0);
        } else if (choice == JOptionPane.NO_OPTION) {
            parent.dispose();
            System.exit(0);
        }
    }

    private static void showFatalError(Exception e) {
        JOptionPane.showMessageDialog(
                null,
                "Критическая ошибка:\n" + e.getMessage(),
                "Ошибка запуска",
                JOptionPane.ERROR_MESSAGE
        );
        System.exit(1);
    }
}

Папка view:
SportSectionGUI:
package org.example.view;

import org.example.db.DatabaseException;
import org.example.db.DatabaseManager;
import org.example.domain.Trainer;

import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.util.List;

public class SportsSectionGUI extends JFrame {
    private DefaultTableModel tableModel;
    private JTable table;
    private JList<String> trainerList;


    // Кнопки управления
    private JButton addSportsmanButton;
    private JButton showAllButton;
    private JButton findSportsmanButton;
    private JButton findProfessionalButton;
    private JButton removeSportsmanButton;
    private JButton addTrainerButton;
    private JButton showTrainerButton;
    private JButton saveButton;
    private JButton loadButton;
    private JButton exitButton;
    private JButton editSportsmanButton;

    public SportsSectionGUI() {
        initializeUI();
        setupTable();
        setupButtons();
        setupLayout();
    }

    private void initializeUI() {
        setTitle("Управление спортивной секцией");
        setSize(1000, 600);
        setDefaultCloseOperation(WindowConstants.DO_NOTHING_ON_CLOSE);
    }

    private void setupTable() {
        tableModel = new DefaultTableModel() {
            @Override
            public boolean isCellEditable(int row, int column) {
                // Разрешаем редактирование всех ячеек, кроме столбца "Тип"
                return column != 0;
            }
        };

        // Настройка столбцов
        tableModel.addColumn("Тип");
        tableModel.addColumn("Имя");
        tableModel.addColumn("Возраст");
        tableModel.addColumn("Вид спорта");
        tableModel.addColumn("Уровень подготовки");
        tableModel.addColumn("Разряд");
        tableModel.addColumn("Стаж");

        table = new JTable(tableModel);
        table.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);
        table.getTableHeader().setReorderingAllowed(false);
    }

    private void setupButtons() {
        // Инициализация кнопок
        addSportsmanButton = new JButton("Добавить спортсмена");
        showAllButton = new JButton("Показать всех");
        findSportsmanButton = new JButton("Найти спортсмена");
        findProfessionalButton = new JButton("Найти профессионала");
        removeSportsmanButton = new JButton("Удалить спортсмена");
        addTrainerButton = new JButton("Добавить тренера");
        showTrainerButton = new JButton("Показать тренера");
        saveButton = new JButton("Сохранить в файл");
        loadButton = new JButton("Загрузить из файла");
        exitButton = new JButton("Выйти");
        editSportsmanButton = new JButton("Редактировать");
    }


    private void setupLayout() {
        // Первая строка кнопок
        JPanel topButtonPanel = new JPanel(new GridLayout(1, 5, 5, 5));
        topButtonPanel.add(addSportsmanButton);
        topButtonPanel.add(showAllButton);
        topButtonPanel.add(findSportsmanButton);
        topButtonPanel.add(findProfessionalButton);
        topButtonPanel.add(removeSportsmanButton);


        // Вторая строка кнопок
        JPanel bottomButtonPanel = new JPanel(new GridLayout(1, 5, 5, 5));
        bottomButtonPanel.add(addTrainerButton);
        bottomButtonPanel.add(showTrainerButton);
        bottomButtonPanel.add(saveButton);
        bottomButtonPanel.add(loadButton);
        bottomButtonPanel.add(exitButton);

        // Объединение панелей
        JPanel buttonPanel = new JPanel(new GridLayout(2, 1, 5, 5));
        buttonPanel.add(topButtonPanel);
        buttonPanel.add(bottomButtonPanel);

        // Основная компоновка
        add(buttonPanel, BorderLayout.NORTH);
        add(new JScrollPane(table), BorderLayout.CENTER);
    }

    public void showError(String message) {
        JOptionPane.showMessageDialog(this, message, "Ошибка", JOptionPane.ERROR_MESSAGE);
    }

    public void updateTableData(Object[][] data) {
        SwingUtilities.invokeLater(() -> {
            tableModel.setRowCount(0);
            for (Object[] row : data) {
                tableModel.addRow(row);
            }
            table.repaint();
        });
    }

    public void updateTableData(Object[][] data, String[] columnNames) {
        SwingUtilities.invokeLater(() -> {
            tableModel.setColumnIdentifiers(columnNames);
            tableModel.setRowCount(0);
            for (Object[] row : data) {
                tableModel.addRow(row);
            }
            table.repaint();
        });
    }

    // Методы для обновления UI
    private void updateTrainersInView() {
        try {
            List<Trainer> trainers = DatabaseManager.getAllTrainers();
            Object[][] data = trainers.stream()
                    .map(t -> new Object[]{
                            t.getName(),
                            t.getSpecialization(),
                            t.getExperienceYears()  // Исправлено с getExp() на getExperienceYears()
                    })
                    .toArray(Object[][]::new);  // Исправлен синтаксис преобразования в массив

            SportsSectionGUI view = null;
            view.updateTableData(data, new String[]{"Имя тренера", "Специализация", "Стаж"}); // Исправлена опечатка "Мия" на "Имя"
        } catch (DatabaseException e) {
            showError("Ошибка загрузки тренеров: " + e.getMessage());
        }
    }

    // Геттеры для кнопок (используются контроллером)
    public JButton getAddSportsmanButton() { return addSportsmanButton; }
    public JButton getShowAllButton() { return showAllButton; }
    public JButton getFindSportsmanButton() { return findSportsmanButton; }
    public JButton getFindProfessionalButton() { return findProfessionalButton; }
    public JButton getRemoveSportsmanButton() { return removeSportsmanButton; }
    public JButton getAddTrainerButton() { return addTrainerButton; }
    public JButton getShowTrainerButton() { return showTrainerButton; }
    public JButton getSaveButton() { return saveButton; }
    public JButton getLoadButton() { return loadButton; }
    public JButton getExitButton() { return exitButton; }
    public JTable getTable() { return table; }
}
Папка dialogs в папке view:
EditDataDialog:
package org.example.view.dialogs;

import javax.swing.*;
import java.awt.*;

public class EditDataDialog extends JDialog {
    private JComboBox<String> typeCombo;
    private JTextField nameField;
    private JTextField ageField;
    private JTextField sportField;
    private JTextField specialField;
    private JTextField expField;

    private boolean confirmed = false;

    public EditDataDialog(JFrame parent, String currentType) {
        super(parent, "Редактирование данных", true);
        initializeUI(currentType);
    }

    private void initializeUI(String currentType) {
        setSize(400, 300);
        setLayout(new BorderLayout());

        typeCombo = new JComboBox<>(new String[]{"Любитель", "Профессионал", "Тренер"});
        typeCombo.setSelectedItem(currentType);

        nameField = new JTextField();
        ageField = new JTextField();
        sportField = new JTextField();
        specialField = new JTextField();
        expField = new JTextField();

        JPanel inputPanel = new JPanel(new GridLayout(6, 2, 5, 5));
        inputPanel.add(new JLabel("Тип:"));
        inputPanel.add(typeCombo);
        inputPanel.add(new JLabel("Имя:"));
        inputPanel.add(nameField);
        inputPanel.add(new JLabel("Возраст:"));
        inputPanel.add(ageField);
        inputPanel.add(new JLabel("Вид спорта/Специализация:"));
        inputPanel.add(sportField);
        inputPanel.add(new JLabel("Уровень/Разряд:"));
        inputPanel.add(specialField);
        inputPanel.add(new JLabel("Стаж (лет):"));
        inputPanel.add(expField);

        JButton okButton = new JButton("Сохранить");
        JButton cancelButton = new JButton("Отмена");

        okButton.addActionListener(e -> {
            confirmed = true;
            dispose();
        });

        cancelButton.addActionListener(e -> dispose());

        JPanel buttonPanel = new JPanel();
        buttonPanel.add(okButton);
        buttonPanel.add(cancelButton);

        add(inputPanel, BorderLayout.CENTER);
        add(buttonPanel, BorderLayout.SOUTH);
        setLocationRelativeTo(getParent());
    }

    public boolean showDialog() {
        setVisible(true);
        return confirmed;
    }

    // Геттеры для данных
    public String getSelectedType() { return (String) typeCombo.getSelectedItem(); }
    public String getName() { return nameField.getText().trim(); }
    public int getAge() {
        try {
            return Integer.parseInt(ageField.getText().trim());
        } catch (NumberFormatException e) {
            return 0;
        }
    }
    public String getSportOrSpecialization() { return sportField.getText().trim(); }
    public String getSpecialValue() { return specialField.getText().trim(); }
    public int getExperience() {
        try {
            return Integer.parseInt(expField.getText().trim());
        } catch (NumberFormatException e) {
            return 0;
        }
    }
}
AddTrainerDialog:
package org.example.view.dialogs;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;

public class AddTrainerDialog extends JDialog {
    private JTextField nameField;
    private JTextField specField;
    private JTextField expField;

    private boolean confirmed = false;

    public AddTrainerDialog(JFrame parent) {
        super(parent, "Добавить тренера", true);
        initializeComponents();
        setupLayout();
        setSize(350, 200);
        setLocationRelativeTo(parent);
        setResizable(false);
    }

    private void initializeComponents() {
        nameField = new JTextField(20);
        specField = new JTextField(20);
        expField = new JTextField(3);
    }

    private void setupLayout() {
        JPanel inputPanel = new JPanel(new GridLayout(3, 2, 5, 5));
        inputPanel.add(new JLabel("Имя:"));
        inputPanel.add(nameField);
        inputPanel.add(new JLabel("Специализация:"));
        inputPanel.add(specField);
        inputPanel.add(new JLabel("Стаж (лет):"));
        inputPanel.add(expField);

        JButton okButton = new JButton("OK");
        okButton.addActionListener(e -> confirm());

        JButton cancelButton = new JButton("Отмена");
        cancelButton.addActionListener(e -> cancel());

        JPanel buttonPanel = new JPanel();
        buttonPanel.add(okButton);
        buttonPanel.add(cancelButton);

        add(inputPanel, BorderLayout.CENTER);
        add(buttonPanel, BorderLayout.SOUTH);
    }

    private void confirm() {
        if (validateInput()) {
            confirmed = true;
            dispose();
        }
    }

    private void cancel() {
        confirmed = false;
        dispose();
    }

    private boolean validateInput() {
        try {
            if (nameField.getText().trim().isEmpty()) {
                throw new IllegalArgumentException("Введите имя тренера");
            }

            if (specField.getText().trim().isEmpty()) {
                throw new IllegalArgumentException("Введите специализацию");
            }

            int exp = Integer.parseInt(expField.getText().trim());
            if (exp < 0 || exp > 70) {
                throw new IllegalArgumentException("Стаж должен быть от 0 до 70 лет");
            }

            return true;
        } catch (NumberFormatException e) {
            JOptionPane.showMessageDialog(this, "Некорректный стаж", "Ошибка", JOptionPane.ERROR_MESSAGE);
            return false;
        } catch (IllegalArgumentException e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Ошибка", JOptionPane.ERROR_MESSAGE);
            return false;
        }
    }

    public boolean showDialog() {
        setVisible(true);
        return confirmed;
    }

    // Геттеры для данных
    public String getName() {
        return nameField.getText().trim();
    }

    public String getSpecialization() {
        return specField.getText().trim();
    }

    public int getExperience() {
        return Integer.parseInt(expField.getText().trim());
    }
}
AddSportsmanDialog:
package org.example.view.dialogs;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;

public class AddSportsmanDialog extends JDialog {
    private JTextField nameField;
    private JTextField ageField;
    private JTextField sportField;
    private JComboBox<String> typeCombo;
    private JLabel dynamicLabel;
    private JComboBox<String> dynamicCombo; // Изменено с JTextField на JComboBox

    private boolean confirmed = false;

    public AddSportsmanDialog(JFrame parent) {
        super(parent, "Добавить спортсмена", true);
        initializeComponents();
        setupLayout();
        setSize(350, 250);
        setLocationRelativeTo(parent);
        setResizable(false);
    }

    private void initializeComponents() {
        nameField = new JTextField(20);
        ageField = new JTextField(3);
        sportField = new JTextField(20);
        typeCombo = new JComboBox<>(new String[]{"Любитель", "Профессионал"});
        dynamicLabel = new JLabel("Уровень подготовки:");

        // Инициализация выпадающего списка с уровнями подготовки
        dynamicCombo = new JComboBox<>(new String[]{"Начинающий", "Средний", "Продвинутый"});

        typeCombo.addActionListener(this::handleTypeChange);
    }

    private void setupLayout() {
        JPanel inputPanel = new JPanel(new GridLayout(5, 2, 5, 5));
        inputPanel.add(new JLabel("Имя:"));
        inputPanel.add(nameField);
        inputPanel.add(new JLabel("Возраст:"));
        inputPanel.add(ageField);
        inputPanel.add(new JLabel("Вид спорта:"));
        inputPanel.add(sportField);
        inputPanel.add(new JLabel("Тип:"));
        inputPanel.add(typeCombo);
        inputPanel.add(dynamicLabel);
        inputPanel.add(dynamicCombo); // Добавляем выпадающий список вместо текстового поля

        JButton okButton = new JButton("OK");
        okButton.addActionListener(e -> confirm());

        JButton cancelButton = new JButton("Отмена");
        cancelButton.addActionListener(e -> cancel());

        JPanel buttonPanel = new JPanel();
        buttonPanel.add(okButton);
        buttonPanel.add(cancelButton);

        add(inputPanel, BorderLayout.CENTER);
        add(buttonPanel, BorderLayout.SOUTH);
    }

    private void handleTypeChange(ActionEvent e) {
        if (typeCombo.getSelectedItem().equals("Любитель")) {
            dynamicLabel.setText("Уровень подготовки:");
            // Устанавливаем варианты для любителей
            dynamicCombo.setModel(new DefaultComboBoxModel<>(new String[]{"Начинающий", "Средний", "Продвинутый"}));
        } else {
            dynamicLabel.setText("Разряд:");
            // Устанавливаем варианты для профессионалов
            dynamicCombo.setModel(new DefaultComboBoxModel<>(new String[]{
                    "3-й юношеский",
                    "2-й юношеский",
                    "1-й юношеский",
                    "3-й взрослый",
                    "2-й взрослый",
                    "1-й взрослый",
                    "КМС",
                    "МС",
                    "МСМК"
            }));
        }
    }

    private void confirm() {
        if (validateInput()) {
            confirmed = true;
            dispose();
        }
    }

    private void cancel() {
        confirmed = false;
        dispose();
    }

    private boolean validateInput() {
        try {
            if (nameField.getText().trim().isEmpty()) {
                throw new IllegalArgumentException("Введите имя спортсмена");
            }

            int age = Integer.parseInt(ageField.getText().trim());
            if (age <= 0 || age > 100) {
                throw new IllegalArgumentException("Возраст должен быть от 1 до 100 лет");
            }

            if (sportField.getText().trim().isEmpty()) {
                throw new IllegalArgumentException("Введите вид спорта");
            }

            return true;
        } catch (NumberFormatException e) {
            JOptionPane.showMessageDialog(this, "Некорректный возраст", "Ошибка", JOptionPane.ERROR_MESSAGE);
            return false;
        } catch (IllegalArgumentException e) {
            JOptionPane.showMessageDialog(this, e.getMessage(), "Ошибка", JOptionPane.ERROR_MESSAGE);
            return false;
        }
    }

    public boolean showDialog() {
        setVisible(true);
        return confirmed;
    }

    // Геттеры для данных
    public String getName() {
        return nameField.getText().trim();
    }

    public int getAge() {
        return Integer.parseInt(ageField.getText().trim());
    }

    public String getSportType() {
        return sportField.getText().trim();
    }

    public String getSportsmanType() {
        return (String) typeCombo.getSelectedItem();
    }

    public String getLevel() {
        return getSportsmanType().equals("Любитель") ? (String) dynamicCombo.getSelectedItem() : "-";
    }

    public String getRank() {
        return getSportsmanType().equals("Профессионал") ? (String) dynamicCombo.getSelectedItem() : "-";
    }
}
папка utils:
SportsFileManager:
package org.example.utils;

import org.example.model.SportSection;
import org.example.domain.Sportsman;
import org.example.domain.Trainer;
import org.example.model.exceptions.TrainerNotFoundException;

import java.io.*;
import java.util.List;

public class SportsFileManager {
    private static final String FILE_EXTENSION = ".dat";

    public static void saveSectionToFile(SportSection section, String fileName) throws IOException {
        String fullPath = ensureFileExtension(fileName);

        try (ObjectOutputStream oos = new ObjectOutputStream(
                new BufferedOutputStream(
                        new FileOutputStream(fullPath)))) {

            // Сохраняем данные в строгом порядке
            oos.writeObject(section.getAllSportsmans());
            oos.writeObject(section.getTrainer());
        } catch (TrainerNotFoundException e) {
            throw new IOException("Ошибка при сохранении тренера", e);
        }
    }

    public static SportSection loadSectionFromFile(String fileName)
            throws IOException, ClassNotFoundException {
        String fullPath = ensureFileExtension(fileName);

        try (ObjectInputStream ois = new ObjectInputStream(
                new BufferedInputStream(
                        new FileInputStream(fullPath)))) {

            SportSection section = new SportSection();

            // Загрузка в том же порядке, что и сохранение
            @SuppressWarnings("unchecked")
            List<Sportsman> sportsmans = (List<Sportsman>) ois.readObject();
            Trainer trainer = (Trainer) ois.readObject();

            // Восстановление состояния
            sportsmans.forEach(s -> {
                try {
                    section.addSportsman(s);
                } catch (Exception e) {
                    System.err.println("Ошибка добавления спортсмена: " + e.getMessage());
                }
            });
            section.setTrainer(trainer);

            return section;
        }
    }

    public static boolean dataFileExists(String fileName) {
        File file = new File(ensureFileExtension(fileName));
        return file.exists() && file.canRead();
    }

    private static String ensureFileExtension(String fileName) {
        return fileName.endsWith(FILE_EXTENSION) ? fileName : fileName + FILE_EXTENSION;
    }
}
FileUtils:
package org.example.utils;

import java.io.File;
import java.io.IOException;


public class FileUtils {


    public static String ensureFileExtension(String fileName, String extension) {
        if (fileName == null || fileName.isEmpty()) {
            throw new IllegalArgumentException("Имя файла не может быть пустым");
        }

        extension = extension.startsWith(".") ? extension : "." + extension;

        return fileName.endsWith(extension) ? fileName : fileName + extension;
    }


    public static boolean isFileWritable(String filePath) {
        File file = new File(filePath);

        if (file.exists()) {
            return file.canWrite();
        } else {
            // Проверяем возможность создания файла
            try {
                return file.createNewFile();
            } catch (IOException e) {
                return false;
            }
        }
    }


    public static String getBaseName(String fileName) {
        int dotIndex = fileName.lastIndexOf('.');
        return (dotIndex == -1) ? fileName : fileName.substring(0, dotIndex);
    }
}

папка model:
SportSection:
package org.example.model;

import org.example.domain.Sportsman;
import org.example.domain.Trainer;
import org.example.domain.Professional;
import org.example.domain.Amateur;
import org.example.model.exceptions.DuplicateSportsmanException;
import org.example.model.exceptions.TrainerNotFoundException;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;


public class SportSection {
    private List<Sportsman> sportsmans;
    private Trainer trainer;

    public SportSection() {
        this.sportsmans = new ArrayList<>();
        this.trainer = null;
    }


    public void addSportsman(Sportsman sportsman) throws DuplicateSportsmanException {
        if (sportsman == null) {
            throw new IllegalArgumentException("Спортсмен не может быть null");
        }

        if (sportsmans.stream().anyMatch(s -> s.getName().equalsIgnoreCase(sportsman.getName()))) {
            throw new DuplicateSportsmanException(
                    "Спортсмен с именем " + sportsman.getName() + " уже существует");
        }

        sportsmans.add(sportsman);
    }

    public void removeSportsman(Sportsman sportsman) {
        sportsmans.remove(sportsman);
    }


    public List<Sportsman> findSportsmanByName(String name) {
        return sportsmans.stream()
                .filter(s -> s.getName().equalsIgnoreCase(name))
                .collect(Collectors.toList());
    }


    public List<Sportsman> findProfessionalByRank(String rank) {
        return sportsmans.stream()
                .filter(s -> s instanceof Professional)
                .map(s -> (Professional) s)
                .filter(p -> p.getRank() != null && p.getRank().equalsIgnoreCase(rank))
                .collect(Collectors.toList());
    }


    public List<Sportsman> getAllSportsmans() {
        return new ArrayList<>(sportsmans);
    }

    public void setTrainer(Trainer trainer) {
        this.trainer = trainer;
    }


    public Trainer getTrainer() throws TrainerNotFoundException {
        if (trainer == null) {
            throw new TrainerNotFoundException("Тренер не назначен");
        }
        return trainer;
    }


    public void replaceSportsman(Sportsman oldSportsman, Sportsman newSportsman) {
        int index = sportsmans.indexOf(oldSportsman);
        if (index != -1) {
            sportsmans.set(index, newSportsman);
        }
    }


    public void replaceAllData(SportSection newSection) {
        if (newSection == null) {
            throw new IllegalArgumentException("Новый объект SportSection не может быть null");
        }
        this.sportsmans = new ArrayList<>(newSection.sportsmans);
        this.trainer = newSection.trainer;
    }


    public int getSportsmanCount() {
        return sportsmans.size();
    }

    public int getProfessionalsCount() {
        return (int) sportsmans.stream()
                .filter(s -> s instanceof Professional)
                .count();
    }


    public int getAmateursCount() {
        return (int) sportsmans.stream()
                .filter(s -> s instanceof Amateur)
                .count();
    }
}
папка Exceptions в папке model:
TrainerNotFoundException:
package org.example.model.exceptions;


public class TrainerNotFoundException extends Exception {
    public TrainerNotFoundException(String message) {
        super(message);
    }
}
DuplicateSportsmanException:
package org.example.model.exceptions;

public class DuplicateSportsmanException extends Exception {
    public DuplicateSportsmanException(String message) {
        super(message);
    }
}
папка domain:
Trainer:
package org.example.domain;

import java.io.Serializable;



public class Trainer implements Serializable {
    private String name;
    private Integer id;
    private String specialization;
    private int experienceYears;

    public Trainer(String name, String specialization, int experienceYears) {
        setName(name);
        setSpecialization(specialization);
        setExperienceYears(experienceYears);
    }

    // Геттеры и сеттеры
    public String getName() {
        return name;
    }

    public void setName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Имя тренера не может быть пустым");
        }
        this.name = name.trim();
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getSpecialization() {
        return specialization;
    }

    public void setSpecialization(String specialization) {
        if (specialization == null || specialization.trim().isEmpty()) {
            throw new IllegalArgumentException("Специализация не может быть пустой");
        }
        this.specialization = specialization.trim();
    }

    public int getExperienceYears() {
        return experienceYears;
    }

    public void setExperienceYears(int experienceYears) {
        if (experienceYears < 0 || experienceYears > 70) {
            throw new IllegalArgumentException("Стаж должен быть от 0 до 70 лет");
        }
        this.experienceYears = experienceYears;
    }

    public void conductTraining() {
        System.out.printf("%s проводит тренировку по %s\n", name, specialization);
    }

    @Override
    public String toString() {
        return String.format("Тренер: %s, Специализация: %s, Стаж: %d лет",
                name, specialization, experienceYears);
    }
}
Sportsman:
package org.example.domain;

import java.io.Serializable;


public abstract class Sportsman implements Serializable {
    private String name;
    private int age;
    private String sportType;

    public Sportsman(String name, int age, String sportType) {
        setName(name);
        setAge(age);
        setSportType(sportType);
    }

    // Геттеры и сеттеры
    public String getName() {
        return name;
    }

    public void setName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Имя не может быть пустым");
        }
        this.name = name.trim();
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age < 5 || age > 100) {
            throw new IllegalArgumentException("Возраст должен быть от 5 до 100 лет");
        }
        this.age = age;
    }

    public String getSportType() {
        return sportType;
    }

    public void setSportType(String sportType) {
        if (sportType == null || sportType.trim().isEmpty()) {
            throw new IllegalArgumentException("Вид спорта не может быть пустым");
        }
        this.sportType = sportType.trim();
    }

    // Абстрактные методы
    public abstract void completeTraining();
    public abstract String getDetails();

    @Override
    public String toString() {
        return String.format("%s, %d лет, %s", name, age, sportType);
    }
}
Professional:
package org.example.domain;

import org.example.domain.enums.Rank;

public class Professional extends Sportsman {
    private Rank rank;
    private String category;

    // Измененный конструктор
    public Professional(String name, int age, String sportType, Rank rank, String category) {
        super(name, age, sportType);
        this.rank = rank;
        this.category = category;
    }

    // Дополнительный конструктор для String rank
    public Professional(String name, int age, String sportType, String rank, String category) {
        super(name, age, sportType);
        this.rank = Rank.fromString(rank);
        this.category = category;
    }

    // Геттеры и сеттеры
    public Rank getRank() {
        return rank;
    }

    public void setRank(Rank rank) {
        this.rank = rank;
    }

    public void setRank(String rank) {
        this.rank = Rank.fromString(rank);
    }

    @Override
    public void completeTraining() {

    }

    @Override
    public String getDetails() {
        return "";
    }
}
Amateur:
package org.example.domain;

import org.example.domain.enums.SkillLevel;

public class Amateur extends Sportsman {
    private SkillLevel skillLevel;

    // Измененный конструктор
    public Amateur(String name, int age, String sportType, SkillLevel skillLevel) {
        super(name, age, sportType);
        this.skillLevel = skillLevel;
    }

    // Дополнительный конструктор для String level
    public Amateur(String name, int age, String sportType, String level) {
        super(name, age, sportType);
        this.skillLevel = SkillLevel.fromString(level);
    }

    // Геттеры и сеттеры
    public SkillLevel getSkillLevel() {
        return skillLevel;
    }

    public void setSkillLevel(SkillLevel skillLevel) {
        this.skillLevel = skillLevel;
    }

    public void setSkillLevel(String level) {
        this.skillLevel = SkillLevel.fromString(level);
    }

    @Override
    public void completeTraining() {

    }

    @Override
    public String getDetails() {
        return "";
    }
}
папка enums в папке domain:
Rank:
package org.example.domain.enums;

public enum Rank {
    YOUTH_3("3-й юношеский"),
    YOUTH_2("2-й юношеский"),
    YOUTH_1("1-й юношеский"),
    ADULT_3("3-й взрослый"),
    ADULT_2("2-й взрослый"),
    ADULT_1("1-й взрослый"),
    CMS("Кандидат в мастера спорта (КМС)"),
    MS("Мастер спорта (МС)"),
    MSIC("Мастер спорта международного класса (МСМК)");

    private final String title;

    Rank(String title) {
        this.title = title;
    }

    public String getTitle() {
        return title;
    }

    public static Rank fromString(String text) {
        for (Rank rank : Rank.values()) {
            if (rank.title.equalsIgnoreCase(text)) {
                return rank;
            }
        }
        throw new IllegalArgumentException("Unknown rank: " + text);
    }

    public boolean equalsIgnoreCase(String rank) {
        return false;
    }
}
SkillLevel:
package org.example.domain.enums;

public enum SkillLevel {
    BEGINNER("Начинающий"),
    INTERMEDIATE("Средний"),
    ADVANCED("Продвинутый");

    private final String description;

    SkillLevel(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }

    public static SkillLevel fromString(String text) {
        for (SkillLevel level : SkillLevel.values()) {
            if (level.description.equalsIgnoreCase(text)) {
                return level;
            }
        }
        throw new IllegalArgumentException("Unknown skill level: " + text);
    }
}
папка db:
DatabaseManager:
package org.example.db;

import org.example.domain.*;
import org.example.domain.enums.Rank;
import org.example.domain.enums.SkillLevel;
import org.example.model.exceptions.DuplicateSportsmanException;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class DatabaseManager {
    private static final String DB_URL = "jdbc:sqlite:sports_section.db";
    private static Connection connection;

    // Initialize database connection
    public static void initialize() throws DatabaseException {
        try {
            connection = DriverManager.getConnection(DB_URL);
            createTablesIfNotExists();
        } catch (SQLException e) {
            throw new DatabaseException("Database connection error", e);
        }
    }

    // В DatabaseManager добавляем новые методы:

    public static void deleteSportsman(String name) throws DatabaseException {
        String sql = "DELETE FROM sportsmen WHERE name = ?";

        try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
            pstmt.setString(1, name);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            throw new DatabaseException("Error deleting sportsman", e);
        }
    }

    public static void updateTrainer(Trainer trainer) throws DatabaseException {
        String sql = "UPDATE trainers SET name = ?, specialization = ?, experience_years = ? WHERE name = ?";

        try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
            pstmt.setString(1, trainer.getName());
            pstmt.setString(2, trainer.getSpecialization());
            pstmt.setInt(3, trainer.getExperienceYears());
            pstmt.setString(4, trainer.getName());
            pstmt.executeUpdate();
        } catch (SQLException e) {
            throw new DatabaseException("Error updating trainer", e);
        }
    }

    private static void createTablesIfNotExists() throws SQLException {
        String sqlTrainers = """
            CREATE TABLE IF NOT EXISTS trainers (
                trainer_id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL UNIQUE,
                specialization TEXT NOT NULL,
                experience_years INTEGER NOT NULL
            )""";

        String sqlSportsmen = """
            CREATE TABLE IF NOT EXISTS sportsmen (
                sportsman_id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL UNIQUE,
                age INTEGER NOT NULL CHECK (age BETWEEN 5 AND 100),
                sport_type TEXT NOT NULL,
                type TEXT NOT NULL CHECK (type IN ('Любитель', 'Профессионал')),
                level TEXT,
                rank TEXT,
                trainer_id INTEGER REFERENCES trainers(trainer_id) ON DELETE SET NULL
            )""";

        try (Statement stmt = connection.createStatement()) {
            stmt.execute(sqlTrainers);
            stmt.execute(sqlSportsmen);
        }
    }

    // ===== Trainer methods =====
    public static List<Trainer> getAllTrainers() throws DatabaseException {
        List<Trainer> trainers = new ArrayList<>();
        String sql = "SELECT * FROM trainers";

        try (Statement stmt = connection.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {

            while (rs.next()) {
                trainers.add(new Trainer(
                        rs.getString("name"),
                        rs.getString("specialization"),
                        rs.getInt("experience_years")
                ));
            }
        } catch (SQLException e) {
            throw new DatabaseException("Error loading trainers", e);
        }
        return trainers;
    }

    public static void deleteTrainer(String name) throws DatabaseException {
        String sql = "DELETE FROM trainers WHERE name = ?";

        try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
            pstmt.setString(1, name);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            throw new DatabaseException("Error deleting trainer", e);
        }
    }

    public static int addTrainer(Trainer trainer) throws DatabaseException {
        String sql = "INSERT INTO trainers (name, specialization, experience_years) VALUES (?, ?, ?)";

        try (PreparedStatement pstmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
            pstmt.setString(1, trainer.getName());
            pstmt.setString(2, trainer.getSpecialization());
            pstmt.setInt(3, trainer.getExperienceYears());
            pstmt.executeUpdate();

            // Получаем ID нового тренера
            try (ResultSet rs = pstmt.getGeneratedKeys()) {
                if (rs.next()) {
                    return rs.getInt(1);
                }
            }
            return -1;
        } catch (SQLException e) {
            throw new DatabaseException("Error adding trainer", e);
        }
    }

    // ===== Sportsman methods =====
    public static List<Sportsman> getAllSportsmen() throws DatabaseException {
        List<Sportsman> sportsmen = new ArrayList<>();
        String sql = "SELECT * FROM sportsmen";

        try (Statement stmt = connection.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {

            while (rs.next()) {
                sportsmen.add(mapSportsman(rs));
            }
        } catch (SQLException e) {
            throw new DatabaseException("Error loading sportsmen", e);
        }
        return sportsmen;
    }

    public static void addSportsman(Sportsman sportsman) throws DatabaseException, DuplicateSportsmanException {
        String sql = "INSERT INTO sportsmen (name, age, sport_type, type, level, rank) VALUES (?, ?, ?, ?, ?, ?)";

        try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
            setSportsmanParameters(pstmt, sportsman);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            if (e.getMessage().contains("UNIQUE constraint failed")) {
                throw new DuplicateSportsmanException("Sportsman with this name already exists");
            }
            throw new DatabaseException("Error adding sportsman", e);
        }
    }

    public static void updateSportsman(Sportsman sportsman) throws DatabaseException {
        String sql = "UPDATE sportsmen SET age = ?, sport_type = ?, type = ?, level = ?, rank = ? WHERE name = ?";

        try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
            setSportsmanParameters(pstmt, sportsman);
            pstmt.setString(6, sportsman.getName());
            pstmt.executeUpdate();
        } catch (SQLException e) {
            throw new DatabaseException("Error updating sportsman", e);
        }
    }

    private static void setSportsmanParameters(PreparedStatement pstmt, Sportsman sportsman) throws SQLException {
        pstmt.setString(1, sportsman.getName());
        pstmt.setInt(2, sportsman.getAge());
        pstmt.setString(3, sportsman.getSportType());

        if (sportsman instanceof Professional) {
            Professional p = (Professional) sportsman;
            pstmt.setString(4, "Профессионал");
            pstmt.setNull(5, Types.VARCHAR);
            pstmt.setString(6, p.getRank().getTitle());
        } else {
            Amateur a = (Amateur) sportsman;
            pstmt.setString(4, "Любитель");
            pstmt.setString(5, a.getSkillLevel().getDescription());
            pstmt.setNull(6, Types.VARCHAR);
        }
    }

    private static Sportsman mapSportsman(ResultSet rs) throws SQLException {
        String name = rs.getString("name");
        int age = rs.getInt("age");
        String sportType = rs.getString("sport_type");
        String type = rs.getString("type");

        if ("Профессионал".equals(type)) {
            String rankStr = rs.getString("rank");
            return new Professional(
                    name,
                    age,
                    sportType,
                    rankStr != null ? rankStr : "", // Обработка NULL значений
                    "Новичок"
            );
        } else {
            String levelStr = rs.getString("level");
            return new Amateur(
                    name,
                    age,
                    sportType,
                    levelStr != null ? levelStr : "" // Обработка NULL значений
            );
        }
    }

    public static void close() {
        try {
            if (connection != null && !connection.isClosed()) {
                connection.close();
            }
        } catch (SQLException e) {
            System.err.println("Error closing connection: " + e.getMessage());
        }
    }
}
DatabaseException:
package org.example.db;

/**
 * Custom exception for database operations
 */
public class DatabaseException extends Exception {
    public DatabaseException(String message) {
        super(message);
    }

    public DatabaseException(String message, Throwable cause) {
        super(message, cause);
    }
}
папка controller:
SportsSectionController:
package org.example.controller;

import org.example.db.DatabaseException;
import org.example.db.DatabaseManager;
import org.example.model.SportSection;
import org.example.model.exceptions.TrainerNotFoundException;
import org.example.view.SportsSectionGUI;
import org.example.domain.*;
import org.example.utils.SportsFileManager;
import org.example.view.dialogs.*;

import javax.swing.*;
import javax.swing.event.ListSelectionEvent;
import javax.swing.event.TableModelEvent;
import java.awt.event.ActionEvent;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class SportsSectionController {
    private final SportSection model;
    private final SportsSectionGUI view;

    public SportsSectionController(SportSection model, SportsSectionGUI view) {
        this.model = model;
        this.view = view;
        initializeController();
        setupEventListeners();
        loadInitialData();
    }

    private void initializeController() {
        setupTableEditors();
        view.getRemoveSportsmanButton().setEnabled(false);
    }

    private void setupEventListeners() {
        view.getAddSportsmanButton().addActionListener(this::handleAddSportsman);
        view.getShowAllButton().addActionListener(e -> showAllSportsmans());
        view.getFindSportsmanButton().addActionListener(this::handleFindSportsman);
        view.getFindProfessionalButton().addActionListener(this::handleFindProfessional);
        view.getRemoveSportsmanButton().addActionListener(this::handleRemoveSportsman);
        view.getAddTrainerButton().addActionListener(this::handleAddTrainer);
        view.getShowTrainerButton().addActionListener(this::handleShowTrainer);
        view.getSaveButton().addActionListener(this::handleSave);
        view.getLoadButton().addActionListener(this::handleLoad);
        view.getExitButton().addActionListener(this::handleExit);
        view.getShowAllButton().addActionListener(e -> updateView());

        view.getTable().getSelectionModel().addListSelectionListener(this::handleTableSelection);
        view.getTable().getModel().addTableModelListener(e -> {
            if (e.getType() == TableModelEvent.UPDATE) {
                handleTableEdit(e.getFirstRow());
            }
        });
    }

    private void setupTableEditors() {
        // Выпадающий список для типа
        JComboBox<String> typeCombo = new JComboBox<>(new String[]{"Любитель", "Профессионал"});
        view.getTable().getColumnModel().getColumn(0).setCellEditor(new DefaultCellEditor(typeCombo));

        // Выпадающий список для уровня подготовки
        JComboBox<String> levelCombo = new JComboBox<>(new String[]{"Начинающий", "Средний", "Продвинутый"});
        view.getTable().getColumnModel().getColumn(4).setCellEditor(new DefaultCellEditor(levelCombo));

        // Выпадающий список для разряда (исправленные значения)
        JComboBox<String> rankCombo = new JComboBox<>(new String[]{
                "3-й юношеский",
                "2-й юношеский",
                "1-й юношеский",
                "3-й взрослый",
                "2-й взрослый",
                "1-й взрослый",
                "КМС",
                "МС",
                "МСМК"
        });
        view.getTable().getColumnModel().getColumn(5).setCellEditor(new DefaultCellEditor(rankCombo));
    }

    private void loadInitialData() {
        try {
            // Загружаем данные из БД при старте
            List<Sportsman> sportsmen = DatabaseManager.getAllSportsmen();
            sportsmen.forEach(s -> {
                try {
                    model.addSportsman(s);
                } catch (Exception e) {
                    System.err.println("Error adding sportsman: " + e.getMessage());
                }
            });

            List<Trainer> trainers = DatabaseManager.getAllTrainers();
            if (!trainers.isEmpty()) {
                model.setTrainer(trainers.get(0)); // Берем первого тренера (можно доработать логику)
            }

            updateView();
        } catch (Exception e) {
            showError("Ошибка загрузки данных из БД: " + e.getMessage());
        }
    }

    private void handleAddSportsman(ActionEvent e) {
        AddSportsmanDialog dialog = new AddSportsmanDialog(view);
        if (dialog.showDialog()) {
            try {
                Sportsman sportsman = createSportsmanFromDialog(dialog);
                DatabaseManager.addSportsman(sportsman);
                model.addSportsman(sportsman);
                updateView();
            } catch (Exception ex) {
                showError(ex.getMessage());
            }
        }
    }

    private Sportsman createSportsmanFromDialog(AddSportsmanDialog dialog) {
        String name = dialog.getName();
        int age = dialog.getAge();
        String sportType = dialog.getSportType();

        if (dialog.getSportsmanType().equals("Любитель")) {
            return new Amateur(name, age, sportType, dialog.getLevel());
        } else {
            return new Professional(name, age, sportType, dialog.getRank(), "-");
        }
    }

    private void handleFindSportsman(ActionEvent e) {
        String name = JOptionPane.showInputDialog(view, "Введите имя спортсмена:");
        if (name != null && !name.trim().isEmpty()) {
            List<Sportsman> found = model.findSportsmanByName(name.trim());
            if (!found.isEmpty()) {
                showFilteredResults(found);
            } else {
                showInfo("Спортсмен не найден");
            }
        }
    }

    private void handleFindProfessional(ActionEvent e) {
        String rank = JOptionPane.showInputDialog(view, "Введите разряд:");
        if (rank != null && !rank.trim().isEmpty()) {
            List<Sportsman> found = model.findProfessionalByRank(rank.trim());
            if (!found.isEmpty()) {
                showFilteredResults(found);
            } else {
                showInfo("Профессионалы с таким разрядом не найдены");
                showAllSportsmans();
            }
        }
    }

    private void handleRemoveSportsman(ActionEvent e) {
        int selectedRow = view.getTable().getSelectedRow();
        if (selectedRow >= 0) {
            String name = (String) view.getTable().getValueAt(selectedRow, 1);
            String type = (String) view.getTable().getValueAt(selectedRow, 0);

            try {
                if ("Тренер".equals(type)) {
                    // Удаляем тренера
                    DatabaseManager.deleteTrainer(name);
                    showInfo("Тренер " + name + " удален");
                } else {
                    // Удаляем спортсмена
                    Sportsman toRemove = model.findSportsmanByName(name).get(0);
                    DatabaseManager.deleteSportsman(name);
                    model.removeSportsman(toRemove);
                }
                updateView();
            } catch (Exception ex) {
                showError("Ошибка удаления: " + ex.getMessage());
            }
        }
    }


    private void handleAddTrainer(ActionEvent e) {
        AddTrainerDialog dialog = new AddTrainerDialog(view);
        if (dialog.showDialog()) {
            try {
                Trainer trainer = new Trainer(
                        dialog.getName(),
                        dialog.getSpecialization(),
                        dialog.getExperience()
                );

                // Добавляем тренера в БД и получаем его ID
                int trainerId = DatabaseManager.addTrainer(trainer);
                trainer.setId(trainerId); // Предполагая, что добавили поле id в класс Trainer
                model.setTrainer(trainer);

                // Обновляем список тренеров в UI
                updateTrainersInView();
                showInfo("Тренер успешно добавлен");
            } catch (Exception ex) {
                showError("Ошибка добавления тренера: " + ex.getMessage());
            }
        }
    }



    private void handleShowTrainer(ActionEvent e) {
        try {
            List<Trainer> trainers = DatabaseManager.getAllTrainers();
            if (!trainers.isEmpty()) {
                Object[][] data = trainers.stream()
                        .map(t -> new Object[]{
                                "Тренер",
                                t.getName(),
                                "-",
                                t.getSpecialization(),
                                "-",
                                "-",
                                t.getExperienceYears()
                        })
                        .toArray(Object[][]::new);

                view.updateTableData(data);
            } else {
                showInfo("Нет зарегистрированных тренеров");
            }
        } catch (Exception ex) {
            showError("Ошибка загрузки тренеров: " + ex.getMessage());
        }
    }

    private void updateTrainersInView() {
        try {
            List<Trainer> trainers = DatabaseManager.getAllTrainers();
            // Обновляем UI с списком тренеров
            // (нужно добавить соответствующий метод в SportsSectionGUI)
        } catch (DatabaseException e) {
            showError("Ошибка загрузки тренеров: " + e.getMessage());
        }
    }

    private void handleSave(ActionEvent e) {
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter("DAT files", "dat"));

        if (fileChooser.showSaveDialog(view) == JFileChooser.APPROVE_OPTION) {
            try {
                SportsFileManager.saveSectionToFile(model, fileChooser.getSelectedFile().getPath());
                showInfo("Данные успешно сохранены");
            } catch (IOException ex) {
                showError("Ошибка сохранения: " + ex.getMessage());
            }
        }
    }

    private void handleLoad(ActionEvent e) {
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter("DAT files", "dat"));

        if (fileChooser.showOpenDialog(view) == JFileChooser.APPROVE_OPTION) {
            try {
                SportSection loaded = SportsFileManager.loadSectionFromFile(fileChooser.getSelectedFile().getPath());
                model.replaceAllData(loaded);
                updateView();
                showInfo("Данные успешно загружены");
            } catch (Exception ex) {
                showError("Ошибка загрузки: " + ex.getMessage());
            }
        }
    }

    private void handleExit(ActionEvent e) {
        int choice = JOptionPane.showConfirmDialog(
                view,
                "Сохранить данные перед выходом?",
                "Подтверждение",
                JOptionPane.YES_NO_CANCEL_OPTION
        );

        if (choice == JOptionPane.YES_OPTION) {
            handleSave(new ActionEvent(this, ActionEvent.ACTION_PERFORMED, "save"));
            System.exit(0);
        } else if (choice == JOptionPane.NO_OPTION) {
            System.exit(0);
        }
    }

    private void handleTableSelection(ListSelectionEvent e) {
        if (!e.getValueIsAdjusting()) {
            boolean hasSelection = view.getTable().getSelectedRow() >= 0;
            view.getRemoveSportsmanButton().setEnabled(hasSelection);
        }
    }

    private void handleTableEdit(int row) {
        try {
            String type = (String) view.getTable().getValueAt(row, 0);
            String name = (String) view.getTable().getValueAt(row, 1);

            List<Sportsman> found = model.findSportsmanByName(name);
            if (!found.isEmpty()) {
                Sportsman sportsman = found.get(0);

                // Обновляем основные данные
                sportsman.setName(name);
                sportsman.setAge(Integer.parseInt(view.getTable().getValueAt(row, 2).toString()));
                sportsman.setSportType((String) view.getTable().getValueAt(row, 3));

                // Обработка для профессионалов
                if (sportsman instanceof Professional) {
                    String rankValue = (String) view.getTable().getValueAt(row, 5);
                    try {
                        ((Professional)sportsman).setRank(rankValue);
                    } catch (IllegalArgumentException e) {
                        // Восстанавливаем предыдущее значение при ошибке
                        view.getTable().setValueAt(((Professional)sportsman).getRank().getTitle(), row, 5);
                        throw e;
                    }
                }
                // Обработка для любителей
                else if (sportsman instanceof Amateur) {
                    String level = (String) view.getTable().getValueAt(row, 4);
                    ((Amateur)sportsman).setSkillLevel(level);
                }

                updateView();
            }
        } catch (Exception ex) {
            showError("Ошибка редактирования: " + ex.getMessage());
            updateView(); // Обновляем таблицу для восстановления корректных значений
        }
    }

    private void showAllSportsmans() {
        updateView();
    }

    private void showFilteredResults(List<Sportsman> sportsmans) {
        Object[][] data = convertToTableData(sportsmans);
        view.updateTableData(data); // Используем базовую версию
    }

    private void updateView() {
        showFilteredResults(model.getAllSportsmans());
    }

    private Object[][] convertToTableData(List<Sportsman> sportsmans) {
        List<Object[]> rows = new ArrayList<>();

        try {
            // Добавляем всех тренеров из базы данных
            List<Trainer> trainers = DatabaseManager.getAllTrainers();
            for (Trainer trainer : trainers) {
                rows.add(new Object[]{
                        "Тренер",
                        trainer.getName(),
                        "-", // Возраст
                        trainer.getSpecialization(),
                        "-", // Уровень
                        "-", // Разряд
                        trainer.getExperienceYears() // Стаж
                });
            }
        } catch (DatabaseException e) {
            showError("Ошибка загрузки тренеров: " + e.getMessage());
        }

        // Добавляем спортсменов
        for (Sportsman s : sportsmans) {
            if (s instanceof Professional) {
                Professional p = (Professional)s;
                rows.add(new Object[]{
                        "Профессионал",
                        p.getName(),
                        p.getAge(),
                        p.getSportType(),
                        "-",
                        p.getRank().getTitle(),
                        "-"
                });
            } else {
                Amateur a = (Amateur)s;
                rows.add(new Object[]{
                        "Любитель",
                        a.getName(),
                        a.getAge(),
                        a.getSportType(),
                        a.getSkillLevel().getDescription(),
                        "-",
                        "-"
                });
            }
        }

        return rows.toArray(new Object[0][]);
    }

    private void showError(String message) {
        JOptionPane.showMessageDialog(view, message, "Ошибка", JOptionPane.ERROR_MESSAGE);
    }

    private void showInfo(String message) {
        JOptionPane.showMessageDialog(view, message, "Информация", JOptionPane.INFORMATION_MESSAGE);
    }
}
