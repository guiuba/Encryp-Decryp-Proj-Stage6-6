# Encryption-Decryption project

My implementation of Encryption-Decryption project stage 6/6 from Jet Brains Academy

Example 1

java Main -mode enc -in road_to_treasure.txt -out protected.txt -key 5 -alg unicode

//This command must get data from the file road_to_treasure.txt, encrypt the data with the key 5, create a file called protected.txt and write ciphertext to it.

Example 2

Input:

java Main -mode enc -key 5 -data "Welcome to hyperskill!" -alg unicode

Output:

\jqhtrj%yt%m~ujwxpnqq&

Example 3

Input:

java Main -key 5 -alg unicode -data "\jqhtrj%yt%m~ujwxpnqq&" -mode dec

Output:

Welcome to hyperskill!

Example 4:

Input:

java Main -key 5 -alg shift -data "Welcome to hyperskill!" -mode enc

Output:

Bjqhtrj yt mdujwxpnqq!

Example 5:

Input:

java Main -key 5 -alg shift -data "Bjqhtrj yt mdujwxpnqq!" -mode dec

Output:

Welcome to hyperskill!

---------------------------------------------------------------
My code:

import java.util.Scanner;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Arrays;
import java.util.Scanner;


public class Main {
    static String newFileName = "";

    public static String setAlg(String[] args) {

        for (int i = 0; i < args.length; i++) {
            if (args[i].equals("-alg")) {
                if (!args[i + 1].startsWith("-")) {
                    return args[i + 1];
                }
            }
        }
        return "shift";
    }

    public static String setData(String[] args) {
        for (int i = 0; i < args.length; i++) {
            if (args[i].equals("-data")) {
                if (!args[i + 1].startsWith("-")) {
                    return args[i + 1];
                }
            }
        }
        return "";
    }

    public static String setMode(String[] args) {
        for (int i = 0; i < args.length; i++) {
            if (args[i].equals("-mode")) {
                if (!args[i + 1].startsWith("-")) {
                    return args[i + 1];
                }
            }
        }
        return "enc";
    }

    public static int setKey(String[] args) {
        for (int i = 0; i < args.length; i++) {
            if (args[i].equals("-key")) {
                if (!args[i + 1].startsWith("-")) {
                    return Integer.parseInt(args[i + 1]);
                }
            }
        }
        return 0;
    }

    public static boolean isOutputToFile(String[] args) {
        for (int i = 0; i < args.length; i++) {
            if (args[i].equals("-out")) {
                if (!args[i + 1].startsWith("-")) {
                    newFileName = args[i + 1];
                    return true;
                }
            }
        }
        return false;
    }

    public static String setInputData(String[] args) {
        for (int i = 0; i < args.length; i++) {
            if (args[i].equals("-in") && !Arrays.toString(args).contains("-data")) {
                if (!args[i + 1].startsWith("-")) {
                    try {

                        return readFileAsString(args[i + 1]);
                    } catch (IOException e) {
                        System.out.println("Error");
                        System.out.println("Cannot read file: " + e.getMessage());
                        break;
                    }
                }
            }
        }
        return null;
    }

    public static String readFileAsString(String fileName) throws IOException {
        return new String(Files.readAllBytes(Paths.get(fileName)));
    }

    public static void main(String[] args) {
        //Scanner scan = new Scanner(System.in); // só para teste, comentar depois
        //String[] args = scan.nextLine().split("\\s+"); // só para teste, comentar depois
        //scan.close(); // só para teste, comentar depois
        String alg = setAlg(args);    // shift(default) or unicode
        String data = setData(args);  // "" (empty string as default)
        String mode = setMode(args);  // enc (default) or dec
        int key = setKey(args);       // 0 (default) or passed arg
        boolean outputToFile = isOutputToFile(args);  // false (default) or passed true
        String inputData = setInputData(args);    // null(default) or passed arg

        if (inputData != null) {
            data = inputData;
        }

        final Alg selectedAlg = selectAlg(alg);

        Context context = new Context();

        context.setAlg(selectedAlg);
        context.invoke(data, mode, key, outputToFile, newFileName);

    }

    public static Alg selectAlg(String alg) {

        switch (alg) {
            case "shift": {
                return new ShiftAlg();
            }
            case "unicode": {
                return new UnicodeAlg();
            }
            default: {
                throw new IllegalArgumentException("Unknown algorithm type " + alg);
            }
        }
    }

}

class Context {
    private Alg alg;

    public void setAlg(Alg alg) {
        this.alg = alg;
    }

    public void invoke(String data, String mode, int key,
                       boolean outputToFile, String newFileName) {
        this.alg.execute(data, mode, key,
                outputToFile, newFileName);
    }
}


interface Alg {

    void execute(String data, String mode, int key, boolean outputToFile,
                 String newFileName);

}

class UnicodeAlg implements Alg {

    public void execute(String data, String mode, int key, boolean outputToFile,
                        String newFileName) {
        String result = "";

        switch (mode) {
            case "enc":
                File file = new File(newFileName);
                for (int i = 0; i < data.length(); i++) {
                    int originalAlphabetPosition = data.charAt(i) - 32;
                    int newAlphabetPosition = (originalAlphabetPosition + key) % 96;
                    result += (char) (newAlphabetPosition + 32);
                }
                if (!outputToFile) {
                    System.out.println(result);
                } else {
                    try (FileWriter writer = new FileWriter(file)) {
                        writer.write(result);
                    } catch (IOException e1) {
                        System.out.println("Error " + e1.getMessage());
                        e1.printStackTrace();
                    }
                }
                break;

            case "dec":
                mode = "enc";
                execute(data, mode, 96 - (key % 96), outputToFile, newFileName);
                break;
            default:
                break;

        }
    }
}

class ShiftAlg implements Alg {

    public void execute(String data, String mode, int key, boolean outputToFile,
                        String newFileName) {
        String result = "";

        switch (mode) {
            case "enc":
                File file = new File(newFileName);
                for (int i = 0; i < data.length(); i++) {
                    if (data.charAt(i) >= 97 && data.charAt(i) <= 122) {
                        int originalAlphabetPosition = data.charAt(i) - 97;
                        int newAlphabetPosition = (originalAlphabetPosition + key) % 26;
                        result += (char) (newAlphabetPosition + 97);

                    } else if (data.charAt(i) >= 65 && data.charAt(i) <= 90) {
                        int originalAlphabetPosition = data.charAt(i) - 65;
                        int newAlphabetPosition = (originalAlphabetPosition + key) % 26;
                        result += (char) (newAlphabetPosition + 65);

                    } else {
                        result += (char) data.charAt(i);
                    }

                }
                if (!outputToFile) {
                    System.out.println(result);
                } else {
                    try (FileWriter writer = new FileWriter(file)) {
                        writer.write(result);
                    } catch (IOException e1) {
                        System.out.println("Error " + e1.getMessage());
                        e1.printStackTrace();
                    }
                }

                break;
            case "dec":
                mode = "enc";
                execute(data, mode, 26 - (key % 26), outputToFile, newFileName);
                break;
        }
    }
}

