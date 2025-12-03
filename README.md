/*
    ----------------------------------------------------------
    Program: Maze Solver
    Course: Intro to C++ (Maze Project)
    Author: Rashaud Luc
    Description:
        The local newspaper wants to publish daily children’s mazes.
        Some automatically generated mazes are broken. This program
        determines whether each maze in maze.dat is solvable.

        A maze:
            - Is 20x20 characters
            - 'S' = Start
            - 'E' = End
            - '#' = Wall
            - '.' = Path
        Movement allowed: UP, DOWN, LEFT, RIGHT (no diagonals)

        The program:
            - Reads each 20x20 maze from maze.dat
            - Stores it in a Maze object
            - Uses BFS to determine if S can reach E
            - Outputs:
                    Maze: YES
              or    Maze: NO

        This file contains the full implementation.
    ----------------------------------------------------------
*/

#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <queue>

using namespace std;

class Maze {
private:
    static const int SIZE = 20;        // Maze is always 20x20
    char grid[SIZE][SIZE];            // Stores maze characters
    bool solvable;                    // TRUE if S → E is reachable

    // Check if coordinates are inside the maze
    bool inBounds(int r, int c) const {
        return r >= 0 && r < SIZE && c >= 0 && c < SIZE;
    }

    // Performs BFS from S to determine if E is reachable
    bool checkSolvable() {
        int startR = -1, startC = -1;
        int endR   = -1, endC   = -1;

        // Find S and E
        for (int r = 0; r < SIZE; r++) {
            for (int c = 0; c < SIZE; c++) {
                if (grid[r][c] == 'S') {
                    startR = r;
                    startC = c;
                }
                else if (grid[r][c] == 'E') {
                    endR = r;
                    endC = c;
                }
            }
        }

        // Safety check (should always be present)
        if (startR == -1 || endR == -1)
            return false;

        // BFS Setup
        bool visited[SIZE][SIZE] = {false};
        queue<pair<int, int>> q;

        q.push({startR, startC});
        visited[startR][startC] = true;

        // Movement: up, right, down, left
        int dr[4] = {-1, 0, 1, 0};
        int dc[4] = {0, 1, 0, -1};

        while (!q.empty()) {
            auto [r, c] = q.front();
            q.pop();

            // Reached the end
            if (r == endR && c == endC)
                return true;

            // Explore neighbors
            for (int i = 0; i < 4; i++) {
                int nr = r + dr[i];
                int nc = c + dc[i];

                if (inBounds(nr, nc) && !visited[nr][nc]) {
                    char tile = grid[nr][nc];

                    // Can step on '.', 'S', or 'E'
                    if (tile != '#') {
                        visited[nr][nc] = true;
                        q.push({nr, nc});
                    }
                }
            }
        }

        return false; // E not reachable
    }

public:
    // Constructor loads the 20 lines into the grid
    Maze(const vector<string>& lines) {
        for (int r = 0; r < SIZE; r++)
            for (int c = 0; c < SIZE; c++)
                grid[r][c] = lines[r][c];

        solvable = checkSolvable();
    }

    // Required output format
    string toString() const {
        return solvable ? "Maze: YES" : "Maze: NO";
    }
};

int main() {
    ifstream file("maze.dat");

    if (!file) {
        cerr << "Error: maze.dat could not be opened.\n";
        return 1;
    }

    vector<string> lines;
    string line;

    while (true) {
        lines.clear();

        // Read until 20 rows are collected (skip blank separators)
        while (lines.size() < 20 && getline(file, line)) {
            if (line.empty()) continue;        // blank separator
            lines.push_back(line.substr(0, 20));
        }

        if (lines.size() != 20) break;         // no more complete mazes

        Maze m(lines);
        cout << m.toString() << endl;
    }

    return 0;
}
