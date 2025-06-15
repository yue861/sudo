#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#define MAX_PROBLEMS 500

// ANSI color codes
#define RESET "\033[0m"
#define RED "\033[31m"
#define GREEN "\033[32m"
#define YELLOW "\033[33m"
#define BLUE "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN "\033[36m"
#define WHITE "\033[37m"
#define BOLD "\033[1m"

int nunber;
// ========== 新增區塊：遊戲用全域變數 ==========
// 說明：增加遊戲所需的基本變數
int player_board[9][9];     // 玩家當前的盤面
int answer_board[9][9];     // 正確答案盤面
int original_board[9][9];   // 原始問題盤面
int error_count = 0;        // 錯誤計數
int input_status[9][9];     // 記錄每個位置的輸入狀態 (0: 初始, 1: 正確, -1: 錯誤)





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

int solve(int puzzle[][9], int pos) {
    // 終止條件：所有位置都填完了
    if (pos == 81) {
        return 1;  // 成功解出
    }

    // 將位置編號轉換為行列座標
    int row = pos / 9;
    int col = pos % 9;

    // 如果該位置已有數字，跳到下一個位置
    if (puzzle[row][col] != 0) {
        return solve(puzzle, pos + 1);
    }

    // 嘗試填入數字 1-9
    for (int num = 1; num <= 9; num++) {
        // 檢查這個數字是否可以放在這個位置
        if (isValid(num, puzzle, row, col)) {
            // 暫時填入這個數字
            puzzle[row][col] = num;

            // 遞迴處理下一個位置
            if (solve(puzzle, pos + 1)) {
                return 1;  // 成功找到解答
            }

            // 如果遞迴失敗，回溯：清空該格
            puzzle[row][col] = 0;
        }
    }

    // 所有數字都試過，仍無法解出
    return 0;
}


int isValid(int number, int puzzle[][9], int row, int col) {
    int rowStart = (row / 3) * 3;
    int colStart = (col / 3) * 3;

    for (int i = 0; i < 9; i++) {
        // 檢查同一行
        if (puzzle[row][i] == number) return 0;

        // 檢查同一列
        if (puzzle[i][col] == number) return 0;

        // 檢查 3x3 小方格
        if (puzzle[rowStart + (i / 3)][colStart + (i % 3)] == number) return 0;
    }

    return 1;
}

// ========== 新增區塊：提示功能 ==========
// 說明：為玩家提供一個正確答案的提示
void give_hint() {
    // 收集所有空的位置
    int empty_positions[81][2];
    int empty_count = 0;

    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                empty_positions[empty_count][0] = i;
                empty_positions[empty_count][1] = j;
                empty_count++;
            }
        }
    }

    if (empty_count == 0) {
        printf("已經沒有空格了！\n");
        return;
    }

    // 隨機選擇一個空位置給提示
    int random_index = rand() % empty_count;
    int hint_row = empty_positions[random_index][0];
    int hint_col = empty_positions[random_index][1];

    // 填入正確答案
    player_board[hint_row][hint_col] = answer_board[hint_row][hint_col];
    input_status[hint_row][hint_col] = 1; // 標記為正確

    printf("💡 提示：第 %d 列第 %d 行應該填 %d\n", 
           hint_row + 1, hint_col + 1, answer_board[hint_row][hint_col]);
}

// ========== 新增區塊：輸入處理函式 ==========
// 說明：處理玩家輸入並檢查答案
int handle_input() {
    int row, col, num;

    printf("請輸入 行 列 數字 (1-9)，或輸入 0 0 0 結束遊戲，輸入 -1 -1 -1 獲得提示: ");
    scanf("%d %d %d", &row, &col, &num);

    // 檢查是否要結束遊戲
    if (row == 0 && col == 0 && num == 0) {
        return -1; // 結束遊戲
    }

    // 檢查是否要獲得提示
    if (row == -1 && col == -1 && num == -1) {
        give_hint();
        return 1; // 繼續遊戲並顯示盤面
    }

    // 檢查輸入範圍
    if (row < 1 || row > 9 || col < 1 || col > 9 || num < 1 || num > 9) {
        printf("輸入超出範圍！請輸入 1-9 之間的數字。\n");
        return 0; // 繼續遊戲
    }

    // 轉換為陣列索引 (0-8)
    row--; col--;

    // 檢查該位置是否為原始數字
    if (original_board[row][col] != 0) {
        printf("該位置是原始數字，不能修改！\n");
        return 0;
    }

    // 檢查該位置是否已經填過正確答案
    if (player_board[row][col] != 0 && input_status[row][col] == 1) {
        printf("該位置已經填過正確答案了！\n");
        return 0;
    }

    // 檢查答案是否正確
    if (answer_board[row][col] == num) {
        // 正確
        player_board[row][col] = num;
        input_status[row][col] = 1; // 標記為正確輸入
        printf(GREEN "正確！" RESET "\n");
        return 1;
    } else {
        // 錯誤
        player_board[row][col] = num;
        input_status[row][col] = -1; // 標記為錯誤輸入
        error_count++;
        printf(RED "錯誤！錯誤次數：%d" RESET "\n", 
error_count);
        return 1;
    }
}
// ========== 新增區塊：遊戲完成檢查 ==========
// 說明：檢查是否完成遊戲
int is_complete() {
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (player_board[i][j] == 0) {
                return 0; // 還有空格
            }
        }
    }
    return 1; // 完成
}

int file_exists(const char *filename) {
    FILE *file = fopen(filename, "rb");
    if (file) {
        fclose(file);
        return 1;
    }
    return 0;
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
                if (original_board[i][j] == board[i][j]) { // If it's original number
                    printf("%d", board[i][j]);
                }
                else if (input_status[i][j] == 1) { // Correct input
                    printf(GREEN "%d" RESET, board[i][j]);
                }
                else if (input_status[i][j] == -1) { // Incorrect input
                    printf(RED "%d" RESET, board[i][j]);
                }
                else {
                    printf("%d", board[i][j]); // Should not happen, but just in case
                }
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
    fclose(fp);

    return 1;
    }

// ========== 新增區塊：遊戲初始化函式 ==========
// 說明：準備遊戲所需的盤面和答案
void init_game(int puzzle[][9]) {
    // 複製原始問題到各個盤面
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            original_board[i][j] = puzzle[i][j];
            player_board[i][j] = puzzle[i][j];
            answer_board[i][j] = puzzle[i][j];
            input_status[i][j] = 0; // 初始化輸入狀態
        }
    }

    // 計算正確答案
    solve(answer_board, 0);

    // 重置錯誤計數
    error_count = 0;

    printf("遊戲初始化完成！\n");
}

// ========== 新增區塊：遊戲主函式 ==========
// 說明：遊戲的主要控制邏輯
void play_game(int puzzle[][9]) {
    printf("=== 數獨遊戲 ===\n");
    printf("規則：輸入 列(橫) 行(直) 數字 來填數字\n");
    printf("輸入 -1 -1 -1 可以獲得提示\n");
    printf("錯誤5次遊戲結束\n\n");

    // 初始化遊戲
    init_game(puzzle);

    // 顯示初始盤面
    printf("初始盤面：\n");
    print_board(puzzle);

    // 遊戲主迴圈
    while (error_count < 5) {
        int result = handle_input();

        if (result == -1) {
            printf("遊戲結束！\n");
            break;
        }

        if (result == 1) {
            // 顯示當前盤面
            printf("\n當前盤面：\n");
            print_board(player_board);

            // 檢查是否完成
            if (is_complete()) {
                printf("🎉 恭喜！你完成了數獨！\n");
                break;
            }
        }
    }

    // 遊戲結束處理
    if (error_count >= 5) {
        printf("💥 錯誤太多次，遊戲結束！\n");
        printf("正確答案：\n");
        print_board(answer_board);
    }
}
#include <stdio.h>
#include <stdlib.h>

// 假設你已有這些函數
int read_from_binary_file(int board[9][9], const char *filename, int index);
void print_board(int board[9][9]);
void play_game(int board[9][9]);

void play_from_database() {
    int number;
    printf("1到100 容易\n101到200 中等\n201到300 困難\n301到400 專家\n請輸入起始題號: ");
    scanf("%d", &number);

    while (number >= 1 && number <= 400) {
        int board[9][9] = {0};
        if (read_from_binary_file(board, "sudoku.bin", number - 1)) {
            printf("\n第 %d 題：\n", number);
            print_board(board);
            play_game(board);

            char choice;
            printf("\n是否繼續下一題？(y/n): ");
            scanf(" %c", &choice);
            if (choice == 'y' || choice == 'Y') {
                number++;
            } else {
                break;
            }
        } else {
            printf("無法讀取第 %d 題\n", number);
            break;
        }
    }

    printf("感謝遊玩！\n");
}

int main() {
srand(time(NULL));
int puzzle[9][9] = {0};
int level, holes;
int i=1, x=0;
const char *filename = "sudoku.bin";

printf("請選擇難度（1: 容易, 2: 中等, 3: 困難, 4: 專家, 5:題庫）: ");
scanf("%d", &level);
if (!file_exists(filename)) {
for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 35);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
    }
    for (int a = 0; a < 100; a++) {
        int puzzle[9][9];
        generate_random_sudoku_unique(puzzle, 45);
        save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
}
    for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 50);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
            }
        for (int a = 0; a < 100; a++) {
            int puzzle[9][9];
            generate_random_sudoku_unique(puzzle, 54);
            save_to_binary_file(puzzle, i + 1, "sudoku.bin", i > 0); // 第一次 is_append=0，其餘為 1
        }
    }   
    switch(level) {
        case 1: holes = 35; break;
        case 2: holes = 45; break;
        case 3: holes = 50; break;
        case 4: holes = 54; break;
        case 5: play_from_database();
            break;
    }
    if (level != 5){
            generate_random_sudoku_unique(puzzle, holes);
            play_game(puzzle);
        }
        return 0;
        
}
