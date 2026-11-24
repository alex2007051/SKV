#include <iostream>
#include <string>

struct Position {
    int x = 0;
    int y = 0;
};

class Entity {
public:
    Entity(Position position, char symbol);

    Position GetPosition() const;
    char GetSymbol() const;

protected:
    void SetPosition(Position position);

private:
    Position position_;
    char symbol_;
};

class Player : public Entity {
public:
    explicit Player(Position position);

    int GetKeys() const;
    unsigned long long GetSteps() const;
    void MoveTo(Position position);
    void CollectKey();
    bool SpendKey();

private:
    int keys_ = 0;
    unsigned long long steps_ = 0;
};

class Player;

class Door : public Entity {
public:
    explicit Door(Position position);

    bool IsOpen() const;
    bool TryOpen(Player& player);

private:
    bool open_ = false;
};

class Maze {
public:
    static constexpr int Width = 11;
    static constexpr int Height = 7;

    Maze();

    bool IsInside(Position position) const;
    char GetCell(Position position) const;
    void RemoveKey(Position position);

private:
    char cells_[Height][Width + 1];
};

class Game {
public:
    int Run();

private:
    enum class Status { Running, Won, Quit };

    const char* HandleCommand(const std::string& command);
    const char* TryMove(Position target);
    const char* StatusText() const;
    void PrintState() const;

    Maze maze_;
    Player player_{{1, 1}};
    Door door_{{5, 3}};
    Status status_ = Status::Running;
};

Entity::Entity(Position position, char symbol)
    : position_(position), symbol_(symbol) {}

Position Entity::GetPosition() const {
    return position_;
}

char Entity::GetSymbol() const {
    return symbol_;
}

void Entity::SetPosition(Position position) {
    position_ = position;
}

Player::Player(Position position) : Entity(position, 'P') {}

int Player::GetKeys() const {
    return keys_;
}

unsigned long long Player::GetSteps() const {
    return steps_;
}

void Player::MoveTo(Position position) {
    SetPosition(position);
    ++steps_;
}

void Player::CollectKey() {
    ++keys_;
}

bool Player::SpendKey() {
    if (keys_ == 0) {
        return false;
    }

    --keys_;
    return true;
}

Door::Door(Position position) : Entity(position, 'D') {}

bool Door::IsOpen() const {
    return open_;
}

bool Door::TryOpen(Player& player) {
    if (open_) {
        return true;
    }

    if (!player.SpendKey()) {
        return false;
    }

    open_ = true;
    return true;
}

Maze::Maze()
    : cells_{
          "###########",
          "#P..#....E#",
          "#.#.#.###.#",
          "#.#..D....#",
          "#.###.###.#",
          "#K....#...#",
          "###########",
      } {
    for (int y = 0; y < Height; ++y) {
        for (int x = 0; x < Width; ++x) {
            char& cell = cells_[y][x];
            if (cell == 'P' || cell == 'D') {
                cell = '.';
            }
        }
    }
}

bool Maze::IsInside(Position position) const {
    return position.x >= 0 && position.x < Width &&
           position.y >= 0 && position.y < Height;
}

char Maze::GetCell(Position position) const {
    return cells_[position.y][position.x];
}

void Maze::RemoveKey(Position position) {
    cells_[position.y][position.x] = '.';
}

namespace {
bool SamePosition(Position first, Position second) {
    return first.x == second.x && first.y == second.y;
}
}

int Game::Run() {
    std::cout << "MAZE_ESCAPE\n";
    PrintState();

    std::string command;
    while (status_ == Status::Running) {
        std::cout << "COMMAND" << std::endl;

        const char* event;
        if (std::getline(std::cin, command)) {
            // Strip CR only if getline consumed a terminating LF.
            if (!std::cin.eof() && !command.empty() && command.back() == '\r') {
                command.pop_back();
            }
            event = HandleCommand(command);
        } else {
            status_ = Status::Quit;
            event = "INPUT_END";
        }

        std::cout << "EVENT " << event << '\n';
        PrintState();
    }

    return 0;
}

const char* Game::HandleCommand(const std::string& command) {
    if (command.size() != 1) {
        return "INVALID_COMMAND";
    }

    const Position current = player_.GetPosition();
    switch (command.front()) {
        case 'W':
        case 'w':
            return TryMove({current.x, current.y - 1});
        case 'A':
        case 'a':
            return TryMove({current.x - 1, current.y});
        case 'S':
        case 's':
            return TryMove({current.x, current.y + 1});
        case 'D':
        case 'd':
            return TryMove({current.x + 1, current.y});
        case 'Q':
        case 'q':
            status_ = Status::Quit;
            return "QUIT";
        default:
            return "INVALID_COMMAND";
    }
}

const char* Game::TryMove(Position target) {
    if (!maze_.IsInside(target)) {
        return "BLOCKED";
    }

    const char cell = maze_.GetCell(target);
    if (cell == '#') {
        return "BLOCKED";
    }

    const bool openingDoor = SamePosition(target, door_.GetPosition()) &&
                             !door_.IsOpen();
    if (openingDoor && !door_.TryOpen(player_)) {
        return "LOCKED";
    }

    player_.MoveTo(target);
    if (openingDoor) {
        return "DOOR_OPENED";
    }

    if (cell == 'K') {
        player_.CollectKey();
        maze_.RemoveKey(target);
        return "KEY_COLLECTED";
    }

    if (cell == 'E') {
        status_ = Status::Won;
        return "WON";
    }

    return "MOVED";
}

const char* Game::StatusText() const {
    switch (status_) {
        case Status::Running:
            return "RUNNING";
        case Status::Won:
            return "WON";
        case Status::Quit:
            return "QUIT";
    }

    return "";
}

void Game::PrintState() const {
    const Position playerPosition = player_.GetPosition();
    const Position doorPosition = door_.GetPosition();
    std::cout << "MAP\n";

    for (int y = 0; y < Maze::Height; ++y) {
        for (int x = 0; x < Maze::Width; ++x) {
            const Position position{x, y};
            if (SamePosition(position, playerPosition)) {
                std::cout << player_.GetSymbol();
            } else if (!door_.IsOpen() && SamePosition(position, doorPosition)) {
                std::cout << door_.GetSymbol();
            } else {
                std::cout << maze_.GetCell(position);
            }
        }
        std::cout << '\n';
    }

    std::cout << "KEYS " << player_.GetKeys() << '\n'
              << "STEPS " << player_.GetSteps() << '\n'
              << "STATUS " << StatusText() << '\n';
}

int main() {
    Game game;
    return game.Run();
}
