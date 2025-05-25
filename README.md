#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500






// 檢查在某格填某個數是否合法
int is_safe(int board[9][9], int row, int col, int num) {
    for (int x = 0; x < 9; x++) {
        if (board[row][x] == num || board[x][col] == num ||
            board[row - row % 3 + x / 3][col - col % 3 + x % 3] == num)
            return 0;
    }
    return 1;
}

// 用回溯法產生完整合法數獨解
int fill_board(int board[9][9], int row, int col) {
    if (row == 9) return 1;
    if (col == 9) return fill_board(board, row + 1, 0);
    if (board[row][col] != 0) return fill_board(board, row, col + 1);

    int numbers[9] = {1,2,3,4,5,6,7,8,9};
    // 打亂數字順序
    for (int i = 8; i > 0; i--) {
        int j = rand() % (i + 1);
        int temp = numbers[i];
        numbers[i] = numbers[j];
        numbers[j] = temp;
    }

    for (int i = 0; i < 9; i++) {
        int num = numbers[i];
        if (is_safe(board, row, col, num)) {
            board[row][col] = num;
            if (fill_board(board, row, col + 1)) return 1;
            board[row][col] = 0;
        }
    }
    return 0;
}

int solve_sudoku(int board[9][9], int count_limit, int* solution_count) {
    if (*solution_count > count_limit) return 0;

    for (int row = 0; row < 9; row++) {
        for (int col = 0; col < 9; col++) {
            if (board[row][col] == 0) {
                for (int num = 1; num <= 9; num++) {
                    if (is_safe(board, row, col, num)) {
                        board[row][col] = num;
                        solve_sudoku(board, count_limit, solution_count);
                        board[row][col] = 0;
                    }
                }
                return 0; // 回溯
            }
        }
    }

    (*solution_count)++; // 找到一組解
    return 1;
}

int has_unique_solution(int board[9][9]) {
    int temp[9][9];
    for (int i = 0; i < 9; i++)
        for (int j = 0; j < 9; j++)
            temp[i][j] = board[i][j];

    int solution_count = 0;
    solve_sudoku(temp, 1, &solution_count);
    return solution_count == 1;
}

void remove_cells_unique(int board[9][9], int num_holes) {
    int attempts = 0;
    int removed = 0;

    while (removed < num_holes && attempts < 1000) {
        int row = rand() % 9;
        int col = rand() % 9;

        if (board[row][col] != 0) {
            int backup = board[row][col];
            board[row][col] = 0;

            if (has_unique_solution(board)) {
                removed++;
            } else {
                board[row][col] = backup; // 還原
            }
            attempts++;
        }
    }

    if (removed < num_holes) {
        printf("⚠️ 僅能安全挖掉 %d 格（要求 %d 格）\n", removed, num_holes);
    }
}

void generate_random_sudoku_unique(int board[9][9], int num_holes) {
    int full[9][9] = {0};
    fill_board(full, 0, 0);

    for (int i = 0; i < 9; i++)
        for (int j = 0; j < 9; j++)
            board[i][j] = full[i][j];

    remove_cells_unique(board, num_holes);
}






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
void print_board(int board[][9]); // Function prototype

int main() {
    srand(time(NULL));
    int puzzle[9][9] = {0};
    int level, holes;

    printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5: 極限, 6: 題庫）: ");
    scanf("%d", &level);

    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: holes = 57; break;
        case 6: if (level == 6) {
                    int nunber;
                    printf("請輸入要選擇的題號: ");
                    scanf("%d",  &nunber);
                    int board[9][9] = {0};
                    if (read_from_binary_file(board, "sudoku.bin", nunber -1)) {
                        printf("\n第 %d 題：\n", nunber);
                        print_board(board);
                    } else {
                        printf("無法讀取題目\n");
                    }
                break;
            default: holes = 45;
            }
        }
        if (level != 6){
            generate_random_sudoku_unique(puzzle, holes);
            print_board(puzzle);
        }
    
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

            if (header.numbers >= MAX_PROBLEMS) {
                printf("⚠️ 題庫已達上限（最多 %d 題）\n", MAX_PROBLEMS);
                fclose(fp);
                return;
            }


            // 自動設定新的題目 ID 為目前數量 + 1
            problem_id = header.numbers + 1;
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

    printf("檔案中有 %d 個數獨問題\n", header.numbers-1);

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
    fclose(fp);

    return 1;
    }
