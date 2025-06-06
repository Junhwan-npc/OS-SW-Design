# OS-SW-Design
2025-1 Open Source-Software Design Project

//Eclipse를 이용해서 Java로 구현

// RunAndHitGUI.java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.*;

public class RunAndHitGUI extends JFrame {
    private JTextField nameField, teamField;
    private JTextArea resultArea;

    public RunAndHitGUI() {
        setTitle("RUN AND HIT - 선수 검색");
        setSize(500, 400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout());

        // 상단 입력 패널
        JPanel inputPanel = new JPanel();
        inputPanel.add(new JLabel("선수 이름:"));
        nameField = new JTextField(10);
        inputPanel.add(nameField);

        inputPanel.add(new JLabel("소속팀:"));
        teamField = new JTextField(10);
        inputPanel.add(teamField);

        JButton searchButton = new JButton("검색");
        inputPanel.add(searchButton);

        add(inputPanel, BorderLayout.NORTH);

        // 결과 출력 창
        resultArea = new JTextArea();
        resultArea.setEditable(false);
        add(new JScrollPane(resultArea), BorderLayout.CENTER);

        // 이벤트 처리
        searchButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String name = nameField.getText().trim();
                String team = teamField.getText().trim();
                displayPlayerInfo(name, team);
            }
        });
    }

    private void displayPlayerInfo(String name, String team) {
        Player player = PlayerDataBase.findPlayer(name, team);
        if (player == null) {
            resultArea.setText("⚠️ 해당 선수는 존재하지 않습니다.");
            return;
        }

        StringBuilder sb = new StringBuilder();
        sb.append("🧾 선수 정보\n");
        sb.append("이름: ").append(player.getName()).append("\n");
        sb.append("포지션: ").append(player.getPosition()).append("\n");
        sb.append("등번호: ").append(player.getNumber()).append("\n");
        sb.append("팀: ").append(player.getTeam()).append("\n\n");

        sb.append("📊 타격 기록\n");
        for (PlayerRecord rec : player.getRecords()) {
            sb.append(rec.getYear()).append("년 | ")
              .append("AVG: ").append(rec.getAvg()).append(", ")
              .append("OBP: ").append(rec.getObp()).append(", ")
              .append("SLG: ").append(rec.getSlg()).append(", ")
              .append("OPS: ").append(rec.getOps()).append("\n");
        }

        resultArea.setText(sb.toString());
    }

    public static void main(String[] args) {
        RunAndHitGUI gui = new RunAndHitGUI();
        gui.setVisible(true);
    }
}

// Player.java
import java.util.*;

public class Player {
    private String name;
    private String position;
    private int number;
    private String team;
    private List<PlayerRecord> records;

    public Player(String name, String position, int number, String team) {
        this.name = name;
        this.position = position;
        this.number = number;
        this.team = team;
        this.records = new ArrayList<>();
    }

    public String getName() { return name; }
    public String getPosition() { return position; }
    public int getNumber() { return number; }
    public String getTeam() { return team; }
    public List<PlayerRecord> getRecords() { return records; }

    public void addRecord(PlayerRecord record) {
        records.add(record);
    }
}


// PlayerRecord.java
public class PlayerRecord {
    private int year;
    private double avg, obp, slg, ops;

    public PlayerRecord(int year, double avg, double obp, double slg) {
        this.year = year;
        this.avg = avg;
        this.obp = obp;
        this.slg = slg;
        this.ops = obp + slg; // OPS = OBP + SLG
    }

    public int getYear() { return year; }
    public double getAvg() { return avg; }
    public double getObp() { return obp; }
    public double getSlg() { return slg; }
    public double getOps() { return ops; }
}


// PlayerDataBase.java
import java.io.*;
import java.util.*;

public class PlayerDataBase {
    private static List<Player> players = new ArrayList<>();

    static {
        loadFromCSV("player.csv");
    }

    public static void loadFromCSV(String filename) {
        Map<String, Player> playerMap = new HashMap<>();

        try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] data = line.split(",");

                if (data.length < 8) continue; // 잘못된 줄 무시

                String name = data[0];
                String position = data[1];
                int number = Integer.parseInt(data[2]);
                String team = data[3];
                int year = Integer.parseInt(data[4]);
                double avg = Double.parseDouble(data[5]);
                double obp = Double.parseDouble(data[6]);
                double slg = Double.parseDouble(data[7]);

                String key = name + "_" + team;

                Player player = playerMap.getOrDefault(key, new Player(name, position, number, team));
                player.addRecord(new PlayerRecord(year, avg, obp, slg));
                playerMap.put(key, player);
            }

            players = new ArrayList<>(playerMap.values());
        } catch (IOException e) {
            System.out.println("⚠️ CSV 파일을 읽을 수 없습니다: " + e.getMessage());
        }
    }

    public static Player findPlayer(String name, String team) {
        for (Player p : players) {
            if (p.getName().equals(name) && p.getTeam().equals(team)) {
                return p;
            }
        }
        return null;
    }
}
