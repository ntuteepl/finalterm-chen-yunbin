#include <iostream>
#include <string>
#include <vector>
#include <cstdlib>
#include <ctime>
#include <algorithm>  // 用於排序
using namespace std;

// 隨機加成函數 (用於隨機加成角色或魔王屬性)
double getRandomFactor(double min, double max) {
    return min + (rand() % 1001) / 1000.0 * (max - min);  // 產生範圍 [min, max] 之間的隨機數
}

// 基本角色類別
class Character {
public:
    string name;
    double intelligence;
    double strength;
    double health;
    double maxHealth;

    Character(string n, double iq, double str, double hp)
        : name(n), intelligence(iq), strength(str), health(hp), maxHealth(hp) {}

    virtual void specialSkill(int turn) {} // 虛擬函數，用於特定技能
    virtual double attack() {
        return strength * 1 + intelligence * 0.5;
    }

    virtual void heal() {
        health += intelligence * 0.2;
        if (health > maxHealth) health = maxHealth;
    }

    bool isAlive() const {
        return health > 0;
    }

    void showStats() {
        cout << name << " - 智商: " << intelligence << ", 攻擊力: " << strength << ", 血量: " << health << "/" << maxHealth << endl;
    }
};

// 勇者類別
class Hero : public Character {
public:
    Hero(string n, double iq, double str, double hp) : Character(n, iq, str, hp) {}

    void specialSkill(int turn) override {
        if (health < maxHealth * 0.1 && isAlive()) {
            cout << name << " 觸發大招：『勇者的反擊』，攻擊力提升 1.5 倍！" << endl;
            strength *= 1.5;
        }
    }
};

// 法師類別
class Mage : public Character {
public:
    Mage(string n, double iq, double str, double hp) : Character(n, iq, str, hp) {}

    void specialSkill(int turn) override {
        if (turn % 5 == 0) {
            cout << name << " 觸發大招：『智慧之光』，智商提升 100%！" << endl;
            intelligence *= 2;  // 智商提升 100%
        }
    }
};

// 王八類別
class Turtle : public Character {
private:
    bool hasRevived; // 記錄是否已經使用過重生技能

public:
    Turtle(string n, double iq, double str, double hp) 
        : Character(n, iq, str, hp), hasRevived(false) {}

    void specialSkill(int turn) override {
        if (!hasRevived && health < maxHealth * 0.1 && isAlive()) {
            cout << name << " 觸發大招：『迴光返照』，血量全數恢復！" << endl;
            health = maxHealth; // 血量恢復到最大值
            hasRevived = true;  // 標記已經使用過重生技能
        }
    }
};

// 魔王類別
class Boss : public Character {
public:
    Boss(string n, double iq, double str, double hp) : Character(n, iq, str, hp) {}

    void specialSkill(int turn) override {}
};

// 魔王1:每三回合攻擊力增加 10
class Boss1 : public Boss {
public:
    Boss1(string n, double str, double hp) : Boss(n, 0, str, hp) {}

    void specialSkill(int turn) override {
        if (turn % 3 == 0) {
            cout << name << " 攻擊力提升 10！" << endl;
            strength += 10;
        }
    }
};

// 魔王4:每兩回合攻擊力增加 10，且每五回合未死回復血量 100
class Boss4 : public Boss {
public:
    Boss4(string n, double str, double hp) : Boss(n, 0, str, hp) {}

    void specialSkill(int turn) override {
        if (turn % 2 == 0) {
            cout << name << " 攻擊力提升 10！" << endl;
            strength += 10;
        }
        if (turn % 5 == 0 && isAlive()) {
            cout << name << " 回復血量 100！" << endl;
            health += 100;
            if (health > maxHealth) health = maxHealth;
        }
    }
};

// 魔王5:攻擊力 80，血量 1500，大招每回合攻擊力提升 10，血量回復 80，每五回合玩家血量降低 100
class Boss5 : public Boss {
public:
    Boss5(string n) : Boss(n, 0, 80, 1500) {}

    void specialSkill(int turn) override {
        strength += 10;
        cout << name << " 每回合攻擊力提升 10，當前攻擊力：" << strength << endl;

        health += 80;
        if (health > maxHealth) health = maxHealth;
        cout << name << " 回復血量 80，當前血量：" << health << endl;

        if (turn % 5 == 0) {
            cout << name << " 每五回合觸發，所有玩家血量降低 100！" << endl;
        }
    }
};

// 隨機加成函數
int getRandom(int min, int max) {
    return rand() % (max - min + 1) + min;
}

// 創建角色函數
Character* createCharacter(string type, int level, int index) {
    string uniqueName = type + to_string(index);  // 給每個角色一個唯一的名稱
    double iq = 30 + 6 * level;
    double str = 50 + 10 * level;
    double hp = 200 + 20 * level;

    double originalIq = iq, originalStr = str, originalHp = hp;

    // 加入隨機加成
    iq *= getRandomFactor(0.9, 1.2);  // 智商隨機加成範圍 [90%, 120%]
    str *= getRandomFactor(0.8, 1.3); // 攻擊力隨機加成範圍 [80%, 130%]
    hp *= getRandomFactor(0.9, 1.2);  // 血量隨機加成範圍 [90%, 120%]

    cout << "角色 " << uniqueName << " 原始屬性：" << endl;
    cout << "智商: " << originalIq << " -> " << iq << ", 攻擊力: " << originalStr << " -> " << str << ", 血量: " << originalHp << " -> " << hp << endl;

    if (type == "勇者") {
        // 勇者攻擊力隨機加成
        str += getRandom(-15, 35);
        return new Hero(uniqueName, iq, str, hp);
    } else if (type == "法師") {
        // 法師智商隨機加成
        iq += getRandom(-10, 40);
        return new Mage(uniqueName, iq, str, hp);
    } else if (type == "王八") {
        // 王八血量隨機加成
        hp += getRandom(-10, 100);
        return new Turtle(uniqueName, iq, str, hp);
    }
    return nullptr;
}

// 創建魔王函數
Boss* createBoss(int bossNumber, char bossIndex) {
    double str = 0, hp = 0;
    switch (bossNumber) {
        case 1: str = 20; hp = 300; break;
        case 2: str = 35; hp = 350; break;
        case 3: str = 50; hp = 550; break;
        case 4: str = 60; hp = 1000; break;
        case 5: str = 80; hp = 1500; break;
        default: return nullptr;
    }

    double originalStr = str, originalHp = hp;

    // 加入隨機加成
    str *= getRandomFactor(0.8, 1.2);  // 攻擊力隨機加成範圍 [80%, 120%]
    hp *= getRandomFactor(0.9, 1.3);   // 血量隨機加成範圍 [90%, 130%]

    cout << "魔王 " << "魔王" << bossIndex << " 原始屬性：" << endl;
    cout << "攻擊力: " << originalStr << " -> " << str << ", 血量: " << originalHp << " -> " << hp << endl;

    string name = "魔王" + string(1, bossIndex);  // 使用A-E編號

    if (bossNumber == 1) return new Boss1(name, str, hp);
    if (bossNumber == 2) return new Boss("魔王2", 0, str, hp);
    if (bossNumber == 3) return new Boss1(name, str, hp);
    if (bossNumber == 4) return new Boss4(name, str, hp);
    if (bossNumber == 5) return new Boss5(name);
    return nullptr;
}

// 遊戲流程
void battle(vector<Character*>& players, vector<Boss*>& bosses) {
    int turn = 1;

    while (true) {
        cout << "=== 第 " << turn << " 回合 ===" << endl;

        // 玩家回合
        for (auto& player : players) {
            if (player->isAlive()) {
                // 找出血量最低的魔王
                Boss* targetBoss = nullptr;
                double minHealth = INFINITY;
                for (auto& boss : bosses) {
                    if (boss->isAlive() && boss->health < minHealth) {
                        minHealth = boss->health;
                        targetBoss = boss;
                    }
                }

                if (targetBoss) {
                    player->specialSkill(turn);
                    double playerDamage = player->attack();
                    targetBoss->health -= playerDamage;
                    cout << player->name << " 對 " << targetBoss->name << " 造成了 " << playerDamage << " 點傷害！" << endl;
                }

                // 玩家回血
                player->heal();
                cout << player->name << " 回復了血量，當前血量：" << player->health << endl;
            }
        }

        // 魔王回合，所有魔王依次攻擊
        for (auto& boss : bosses) {
            if (boss->isAlive()) {
                // 找出血量最低的角色
                Character* targetPlayer = nullptr;
                double minHealth = INFINITY;
                for (auto& player : players) {
                    if (player->isAlive() && player->health < minHealth) {
                        minHealth = player->health;
                        targetPlayer = player;
                    }
                }

                if (targetPlayer) {
                    double bossDamage = boss->attack();
                    targetPlayer->health -= bossDamage;
                    cout << boss->name << " 對 " << targetPlayer->name << " 造成了 " << bossDamage << " 點傷害！" << endl;
                }
            }
        }

        // 顯示所有角色的血量
        for (auto& player : players) {
            player->showStats();
        }
        for (auto& boss : bosses) {
            boss->showStats();
        }

        // 檢查玩家是否死亡
        vector<string> deadPlayers;
        for (auto& player : players) {
            if (!player->isAlive()) {
                deadPlayers.push_back(player->name);
            }
        }

        // 檢查魔王是否死亡
        vector<string> deadBosses;
        for (auto& boss : bosses) {
            if (!boss->isAlive()) {
                deadBosses.push_back(boss->name);
            }
        }

        if (!deadPlayers.empty()) {
            cout << "玩家死亡: ";
            for (const string& name : deadPlayers) cout << name << " ";
            cout << endl;
        }

        if (!deadBosses.empty()) {
            cout << "魔王死亡: ";
            for (const string& name : deadBosses) cout << name << " ";
            cout << endl;
        }

        // 檢查遊戲結束條件
        if (deadPlayers.size() == players.size()) {
            cout << "玩家全軍覆沒，遊戲結束！" << endl;
            break;
        }
        if (deadBosses.size() == bosses.size()) {
            cout << "所有魔王死亡，玩家獲勝！" << endl;
            break;
        }

        turn++;
    }
}

int main() {
    srand(time(0));  // 設置隨機種子

    int numPlayers, level;
    string characterType;

    cout << "請輸入玩家數量（最多 5 位玩家）: ";
    cin >> numPlayers;
    if (numPlayers < 1 || numPlayers > 5) {
        cout << "玩家數量必須在 1 到 5 之間！遊戲結束。" << endl;
        return 0;
    }

    vector<Character*> players;

    // 創建玩家
    for (int i = 0; i < numPlayers; i++) {
        cout << "請選擇角色類型（勇者、法師、王八）: ";
        cin >> characterType;

        cout << "請輸入玩家的等級（1-10）: ";
        cin >> level;
        if (level < 1 || level > 10) {
            cout << "等級必須在 1 到 10 之間！遊戲結束。" << endl;
            return 0;
        }

        players.push_back(createCharacter(characterType, level, i+1));
    }

    // 創建魔王數量
    int numBosses;
    cout << "請輸入魔王數量（最多 5 位魔王）：";
    cin >> numBosses;
    if (numBosses < 1 || numBosses > 5) {
        cout << "魔王數量必須在 1 到 5 之間！遊戲結束。" << endl;
        return 0;
    }

    vector<Boss*> bosses;

    // 創建魔王
    for (int i = 0; i < numBosses; i++) {
        cout << "請選擇魔王類型（1-5）: ";
        int bossType;
        cin >> bossType;

        char bossIndex = 'A' + i;
        bosses.push_back(createBoss(bossType, bossIndex));
    }

    // 開始戰鬥
    battle(players, bosses);

    // 清理記憶體
    for (auto& player : players) {
        delete player;
    }
    for (auto& boss : bosses) {
        delete boss;
    }

    return 0;
}
