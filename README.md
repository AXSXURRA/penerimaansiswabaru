import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.*;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class Main {
    private JFrame frame;
    private JTable table;
    private DefaultTableModel tableModel;
    private JTextField txtNama, txtNIS, txtNilaiBIndo, txtNilaiMatematika, txtNilaiPancasila;
    private JComboBox<String> comboJurusan;
    private JButton btnSimpan, btnHapus, btnCari, btnSortNama, btnSortNilai;
    private JTextField txtCari;
    private List<Siswa> dataSiswa = new ArrayList<>();
    private static final String FILE_NAME = "data_siswa.txt";

    public Main() {
        initialize();
        loadDataFromFile();
    }

    private void initialize() {
        frame = new JFrame("Aplikasi Penerimaan Siswa Baru - Admin");
        frame.setSize(900, 600);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLayout(new BorderLayout());

        // Panel input
        JPanel inputPanel = new JPanel(new GridLayout(7, 2, 5, 5));
        inputPanel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

        inputPanel.add(new JLabel("Nama:"));
        txtNama = new JTextField();
        inputPanel.add(txtNama);

        inputPanel.add(new JLabel("NIS:"));
        txtNIS = new JTextField();
        inputPanel.add(txtNIS);

        inputPanel.add(new JLabel("Jurusan:"));
        String[] jurusanOptions = {"Teknik Informatika", "Teknik Industri", "Pertanian"};
        comboJurusan = new JComboBox<>(jurusanOptions);
        inputPanel.add(comboJurusan);

        inputPanel.add(new JLabel("Nilai Bahasa Indonesia:"));
        txtNilaiBIndo = new JTextField();
        inputPanel.add(txtNilaiBIndo);

        inputPanel.add(new JLabel("Nilai Matematika:"));
        txtNilaiMatematika = new JTextField();
        inputPanel.add(txtNilaiMatematika);

        inputPanel.add(new JLabel("Nilai Pancasila:"));
        txtNilaiPancasila = new JTextField();
        inputPanel.add(txtNilaiPancasila);

        // Panel button
        JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.CENTER, 10, 10));
        btnSimpan = new JButton("Simpan");
        btnHapus = new JButton("Hapus");
        buttonPanel.add(btnSimpan);
        buttonPanel.add(btnHapus);

        // Panel pencarian
        JPanel searchPanel = new JPanel(new BorderLayout(5, 5));
        searchPanel.add(new JLabel("Cari (Nama/NIS):"), BorderLayout.WEST);
        txtCari = new JTextField();
        searchPanel.add(txtCari, BorderLayout.CENTER);
        btnCari = new JButton("Cari");
        searchPanel.add(btnCari, BorderLayout.EAST);

        // Panel sorting
        JPanel sortPanel = new JPanel(new FlowLayout(FlowLayout.CENTER, 10, 10));
        btnSortNama = new JButton("Sortir Berdasar Nama");
        btnSortNilai = new JButton("Sortir Berdasar Nilai Rata-rata");
        sortPanel.add(btnSortNama);
        sortPanel.add(btnSortNilai);

        // Panel atas (input + button)
        JPanel topPanel = new JPanel(new BorderLayout());
        topPanel.add(inputPanel, BorderLayout.CENTER);
        topPanel.add(buttonPanel, BorderLayout.SOUTH);

        // Panel utara (top + search + sort)
        JPanel northPanel = new JPanel(new BorderLayout());
        northPanel.add(topPanel, BorderLayout.NORTH);
        northPanel.add(searchPanel, BorderLayout.CENTER);
        northPanel.add(sortPanel, BorderLayout.SOUTH);

        frame.add(northPanel, BorderLayout.NORTH);

        // Table
        String[] columnNames = {"Nama", "NIS", "Jurusan", "B. Indo", "Matematika", "Pancasila", "Rata-rata", "Keterangan"};
        tableModel = new DefaultTableModel(columnNames, 0) {
            @Override
            public boolean isCellEditable(int row, int column) {
                return false;
            }
        };
        table = new JTable(tableModel);
        JScrollPane scrollPane = new JScrollPane(table);
        frame.add(scrollPane, BorderLayout.CENTER);

        // Event listeners
        btnSimpan.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                simpanData();
            }
        });

        btnHapus.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                hapusData();
            }
        });

        btnCari.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                cariData();
            }
        });

        btnSortNama.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                sortByNama();
            }
        });

        btnSortNilai.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                sortByNilai();
            }
        });

        table.getSelectionModel().addListSelectionListener(e -> {
            if (!e.getValueIsAdjusting() && table.getSelectedRow() != -1) {
                int selectedRow = table.getSelectedRow();
                txtNama.setText(table.getValueAt(selectedRow, 0).toString());
                txtNIS.setText(table.getValueAt(selectedRow, 1).toString());
                comboJurusan.setSelectedItem(table.getValueAt(selectedRow, 2).toString());
                txtNilaiBIndo.setText(table.getValueAt(selectedRow, 3).toString());
                txtNilaiMatematika.setText(table.getValueAt(selectedRow, 4).toString());
                txtNilaiPancasila.setText(table.getValueAt(selectedRow, 5).toString());
            }
        });

        frame.setVisible(true);
    }

    private void simpanData() {
        try {
            String nama = txtNama.getText().trim();
            String nis = txtNIS.getText().trim();
            String jurusan = comboJurusan.getSelectedItem().toString();
            double nilaiBIndo = Double.parseDouble(txtNilaiBIndo.getText());
            double nilaiMatematika = Double.parseDouble(txtNilaiMatematika.getText());
            double nilaiPancasila = Double.parseDouble(txtNilaiPancasila.getText());

            if (nama.isEmpty() || nis.isEmpty()) {
                JOptionPane.showMessageDialog(frame, "Nama dan NIS tidak boleh kosong!", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }

            double rataRata = (nilaiBIndo + nilaiMatematika + nilaiPancasila) / 3;
            String keterangan = rataRata >= 70 ? "Lulus" : "Tidak Lulus";

            // Check if this is an existing student (by NIS)
            boolean found = false;
            for (Siswa siswa : dataSiswa) {
                if (siswa.getNis().equals(nis)) {
                    // Update existing student
                    siswa.setNama(nama);
                    siswa.setJurusan(jurusan);
                    siswa.setNilaiBIndo(nilaiBIndo);
                    siswa.setNilaiMatematika(nilaiMatematika);
                    siswa.setNilaiPancasila(nilaiPancasila);
                    siswa.setRataRata(rataRata);
                    siswa.setKeterangan(keterangan);
                    found = true;
                    break;
                }
            }

            if (!found) {
                // Add new student
                Siswa siswa = new Siswa(nama, nis, jurusan, nilaiBIndo, nilaiMatematika, nilaiPancasila, rataRata, keterangan);
                dataSiswa.add(siswa);
            }

            updateTable();
            clearForm();
            saveDataToFile();
        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(frame, "Nilai harus berupa angka!", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void hapusData() {
        if (table.getSelectedRow() == -1) {
            JOptionPane.showMessageDialog(frame, "Pilih data yang akan dihapus!", "Peringatan", JOptionPane.WARNING_MESSAGE);
            return;
        }

        int confirm = JOptionPane.showConfirmDialog(frame, "Apakah Anda yakin ingin menghapus data ini?", "Konfirmasi", JOptionPane.YES_NO_OPTION);
        if (confirm == JOptionPane.YES_OPTION) {
            String nis = txtNIS.getText();
            dataSiswa.removeIf(siswa -> siswa.getNis().equals(nis));
            updateTable();
            clearForm();
            saveDataToFile();
        }
    }

    private void cariData() {
        String keyword = txtCari.getText().trim().toLowerCase();
        if (keyword.isEmpty()) {
            updateTable();
            return;
        }

        List<Siswa> hasilPencarian = new ArrayList<>();
        for (Siswa siswa : dataSiswa) {
            if (siswa.getNama().toLowerCase().contains(keyword) || siswa.getNis().toLowerCase().contains(keyword)) {
                hasilPencarian.add(siswa);
            }
        }

        updateTable(hasilPencarian);
    }

    private void sortByNama() {
        Collections.sort(dataSiswa, new Comparator<Siswa>() {
            @Override
            public int compare(Siswa s1, Siswa s2) {
                return s1.getNama().compareToIgnoreCase(s2.getNama());
            }
        });
        updateTable();
    }

    private void sortByNilai() {
        Collections.sort(dataSiswa, new Comparator<Siswa>() {
            @Override
            public int compare(Siswa s1, Siswa s2) {
                return Double.compare(s2.getRataRata(), s1.getRataRata()); // Descending
            }
        });
        updateTable();
    }

    private void updateTable() {
        updateTable(dataSiswa);
    }

    private void updateTable(List<Siswa> list) {
        tableModel.setRowCount(0);
        for (Siswa siswa : list) {
            Object[] rowData = {
                    siswa.getNama(),
                    siswa.getNis(),
                    siswa.getJurusan(),
                    siswa.getNilaiBIndo(),
                    siswa.getNilaiMatematika(),
                    siswa.getNilaiPancasila(),
                    String.format("%.2f", siswa.getRataRata()),
                    siswa.getKeterangan()
            };
            tableModel.addRow(rowData);
        }
    }

    private void clearForm() {
        txtNama.setText("");
        txtNIS.setText("");
        comboJurusan.setSelectedIndex(0);
        txtNilaiBIndo.setText("");
        txtNilaiMatematika.setText("");
        txtNilaiPancasila.setText("");
    }

    private void saveDataToFile() {
        try (PrintWriter writer = new PrintWriter(new FileWriter(FILE_NAME))) {
            for (Siswa siswa : dataSiswa) {
                writer.println(siswa.toFileString());
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(frame, "Gagal menyimpan data ke file!", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void loadDataFromFile() {
        dataSiswa.clear();
        File file = new File(FILE_NAME);
        if (file.exists()) {
            try (BufferedReader reader = new BufferedReader(new FileReader(FILE_NAME))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    Siswa siswa = Siswa.fromFileString(line);
                    if (siswa != null) {
                        dataSiswa.add(siswa);
                    }
                }
                updateTable();
            } catch (IOException e) {
                JOptionPane.showMessageDialog(frame, "Gagal membaca data dari file!", "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new Main());
    }
}

class Siswa {
    private String nama;
    private String nis;
    private String jurusan;
    private double nilaiBIndo;
    private double nilaiMatematika;
    private double nilaiPancasila;
    private double rataRata;
    private String keterangan;

    public Siswa(String nama, String nis, String jurusan, double nilaiBIndo, double nilaiMatematika,
                 double nilaiPancasila, double rataRata, String keterangan) {
        this.nama = nama;
        this.nis = nis;
        this.jurusan = jurusan;
        this.nilaiBIndo = nilaiBIndo;
        this.nilaiMatematika = nilaiMatematika;
        this.nilaiPancasila = nilaiPancasila;
        this.rataRata = rataRata;
        this.keterangan = keterangan;
    }

    // Getter dan Setter
    public String getNama() {
        return nama;
    }

    public void setNama(String nama) {
        this.nama = nama;
    }

    public String getNis() {
        return nis;
    }

    public void setNis(String nis) {
        this.nis = nis;
    }

    public String getJurusan() {
        return jurusan;
    }

    public void setJurusan(String jurusan) {
        this.jurusan = jurusan;
    }

    public double getNilaiBIndo() {
        return nilaiBIndo;
    }

    public void setNilaiBIndo(double nilaiBIndo) {
        this.nilaiBIndo = nilaiBIndo;
    }

    public double getNilaiMatematika() {
        return nilaiMatematika;
    }

    public void setNilaiMatematika(double nilaiMatematika) {
        this.nilaiMatematika = nilaiMatematika;
    }

    public double getNilaiPancasila() {
        return nilaiPancasila;
    }

    public void setNilaiPancasila(double nilaiPancasila) {
        this.nilaiPancasila = nilaiPancasila;
    }

    public double getRataRata() {
        return rataRata;
    }

    public void setRataRata(double rataRata) {
        this.rataRata = rataRata;
    }

    public String getKeterangan() {
        return keterangan;
    }

    public void setKeterangan(String keterangan) {
        this.keterangan = keterangan;
    }

    public String toFileString() {
        return nama + " " +
                "=" + nis + "=" + jurusan + "=" + nilaiBIndo + "=" +
                nilaiMatematika + "=" + nilaiPancasila + "=" + rataRata + "=" + keterangan;
    }

    public static Siswa fromFileString(String line) {
        try {
            String[] parts = line.split(";");
            String nama = parts[0];
            String nis = parts[1];
            String jurusan = parts[2];
            double nilaiBIndo = Double.parseDouble(parts[3]);
            double nilaiMatematika = Double.parseDouble(parts[4]);
            double nilaiPancasila = Double.parseDouble(parts[5]);
            double rataRata = Double.parseDouble(parts[6]);
            String keterangan = parts[7];

            return new Siswa(nama, nis, jurusan, nilaiBIndo, nilaiMatematika, nilaiPancasila, rataRata, keterangan);
        } catch (Exception e) {
            return null;
        }
    }
}
