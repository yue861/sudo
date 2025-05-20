#include <stdio.h>
int board[9][9] = {
    {0, 0, 0, 0, 0, 0, 0, 9, 0},
    {1, 9, 0, 4, 7, 0, 6, 0, 8},
    {0, 5, 2, 8, 1, 9, 4, 0, 7},
    {2, 0, 0, 0, 4, 8, 0, 0, 0},
    {0, 0, 9, 0, 0, 0, 5, 0, 0},
    {0, 0, 0, 7, 5, 0, 0, 0, 9},
    {9, 0, 7, 3, 6, 4, 1, 8, 0},
    {5, 0, 6, 0, 8, 1, 0, 7, 4},
    {0, 8, 0, 0, 0, 0, 0, 0, 0}
};

typedef struct {
    int numbers;   // 檔案中的問題總數
    int datasize;  // 每個問題的資料大小（位元組）
} SudokuDataHeader;

typedef struct {
    int id;         // 問題編號
    int data[9][9]; // 盤面資料
} SudokuProblem;
int problem_id = 1; // 問題編號
void save_to_binary_file(int board[][9], int problem_id, const char* filename, int is_append); // Function prototype
void save_to_text_file(int board[][9], const char* filename); // Function prototype
void print_board(int board[][9]); // Function prototype

int main()
{

    // 儲存到文本檔案
    save_to_text_file(board, "sudoku.txt");

    // 讀取文本檔案
    int new_board[9][9] = {0};
    if (read_from_text_file(new_board, "sudoku.txt")) {
        printf("\n從文本檔案讀取的數獨盤面：\n");
        print_board(new_board);
    }

    // 儲存到二進位檔案
    save_to_binary_file(board, problem_id, "sudoku.bin", 0);

    // 讀取二進位檔案
    int binary_board[9][9] = {0};
    if (read_from_binary_file(binary_board, "sudoku.bin", 0)) {
        printf("\n從二進位檔案讀取的數獨盤面：\n");
        print_board(binary_board);
    }

    return 0;
}
void print_board(int board[][9]) {
    int i, j;
    printf("\n +-------+-------+-------+\n");
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (j % 3 == 0) printf(" | ");
            else printf(" ");
            if (board[i][j] == 0) {
                printf("_");
}
            else {
                printf("%d", board[i][j]);
            }
        }
        printf(" |\n");
        if (i % 3 == 2) printf(" +-------+-------+-------+\n");
    }
}

// Function to save Sudoku board to a text file
void save_to_text_file(int board[][9], const char* filename) {
    FILE *fp = fopen(filename, "w");
    if (fp == NULL) {
        printf("無法開啟檔案 %s 進行寫入\n", filename);
        return;
    }

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (board[i][j] == 0) {
                fprintf(fp, ".");  // 使用點表示空格
            } else {
                fprintf(fp, "%d", board[i][j]);
            }
        }
        fprintf(fp, "\n");  // 每行結束換行
    }

    fclose(fp);
    printf("已成功儲存到 %s\n", filename);
}

// Function to read Sudoku board from a text file
int read_from_text_file(int board[][9], const char* filename) {
    FILE *fp = fopen(filename, "r");
    if (fp == NULL) {
        printf("無法開啟檔案 %s 進行讀取\n", filename);
        return 0;
    }

    char line[20];  // 足夠容納一行的緩衝區
    int row = 0;

    while (row < 9 && fgets(line, sizeof(line), fp) != NULL) {
        for (int col = 0; col < 9; col++) {
            if (line[col] == '.') {
                board[row][col] = 0;  // 空格
            } else if (line[col] >= '1' && line[col] <= '9') {
                board[row][col] = line[col] - '0';  // 轉換字元到數字
            } else {
                continue;  // 忽略其他字元（如換行）
            }
        }
        row++;
    }

    fclose(fp);

    if (row < 9) {
        printf("警告：檔案格式不正確或檔案不完整\n");
        return 0;
    }

    printf("已成功從 %s 讀取數獨盤面\n", filename);
    return 1;
}




void save_to_binary_file(int board[][9], int problem_id, const char* filename, int is_append) {
    FILE *fp;
    if (is_append) {
        // 檢查檔案是否存在
        fp = fopen(filename, "rb+");
        if (fp == NULL) {
            // 檔案不存在，建立新檔案
            fp = fopen(filename, "wb+");
            if (fp == NULL) {
                printf("無法建立檔案 %s\n", filename);
                return;
            }

            // 寫入新的標頭
            SudokuDataHeader header;
            header.numbers = 1;
            header.datasize = sizeof(SudokuProblem);
            fwrite(&header, sizeof(header), 1, fp);
        } else {
            // 檔案存在，更新標頭中的問題數量
            SudokuDataHeader header;
            fread(&header, sizeof(header), 1, fp);
            header.numbers++;

            // 回到檔案開頭更新標頭
            fseek(fp, 0, SEEK_SET);
            fwrite(&header, sizeof(header), 1, fp);

            // 移動到檔案末尾以添加新問題
            fseek(fp, 0, SEEK_END);
        }
    } else {
        // 建立新檔案
        fp = fopen(filename, "wb");
        if (fp == NULL) {
            printf("無法開啟檔案 %s 進行寫入\n", filename);
            return;
        }

        // 寫入標頭
        SudokuDataHeader header;
        header.numbers = 1;
        header.datasize = sizeof(SudokuProblem);
        fwrite(&header, sizeof(header), 1, fp);
    }

    // 建立並寫入問題
    SudokuProblem problem;
    problem.id = problem_id;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            problem.data[i][j] = board[i][j];
        }
    }

    fwrite(&problem, sizeof(problem), 1, fp);
    fclose(fp);

    printf("已成功儲存到二進位檔案 %s\n", filename);
}

// Function to save Sudoku board to a binary file
int read_from_binary_file(int board[][9], const char* filename, int problem_index) {
    FILE *fp = fopen(filename, "rb");
    if (fp == NULL) {
        printf("無法開啟檔案 %s 進行讀取\n", filename);
        return 0;
    }

    // 讀取標頭
    SudokuDataHeader header;
    fread(&header, sizeof(header), 1, fp);

    printf("檔案中有 %d 個數獨問題\n", header.numbers);

    if (problem_index < 0 || problem_index >= header.numbers) {
        printf("問題編號 %d 超出範圍 (0-%d)\n", problem_index, header.numbers - 1);
        fclose(fp);
        return 0;
    }

    // 跳到指定的問題位置
    fseek(fp, sizeof(header) + problem_index * header.datasize, SEEK_SET);

    // 讀取問題
    SudokuProblem problem;
    fread(&problem, sizeof(problem), 1, fp);

    // 將問題資料複製到提供的板盤中
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            board[i][j] = problem.data[i][j];
        }
    }

    printf("已讀取問題 ID: %d\n", problem.id);
    fclose(fp);

    return 1;
}
