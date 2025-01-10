#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

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
            cout << name << " 觸發大招：『智慧之光』，智商提升 10%！" << endl;
            intelligence *= 1.1;
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

// 創建角色函數
Character* createCharacter(string type, int level, int index) {
    string uniqueName = type + to_string(index);  // 給每個角色一個唯一的名稱

    if (type == "勇者") {
        return new Hero(uniqueName, 30 + 6 * level, 50 + 10 * level, 200 + 20 * level);
    } else if (type == "法師") {
        return new Mage(uniqueName, 50 + 10 * level, 30 + 6 * level, 175 + 15 * level);
    } else if (type == "王八") {
        return new Turtle(uniqueName, 40 + 8 * level, 40 + 8 * level, 250 + 30 * level);
    }
    return nullptr;
}

// 創建魔王函數
Boss* createBoss(int bossNumber) {
    if (bossNumber == 1) return new Boss1("魔王1", 20, 300);
    if (bossNumber == 2) return new Boss("魔王2", 0, 35, 350);
    if (bossNumber == 3) return new Boss1("魔王3", 50, 550);
    if (bossNumber == 4) return new Boss4("魔王4", 60, 1000);
    if (bossNumber == 5) return new Boss5("魔王5");
    return nullptr;
}

// 遊戲流程
void battle(vector<Character*>& players, Boss& boss) {
    int turn = 1;

    while (true) {
        cout << "=== 第 " << turn << " 回合 ===" << endl;

        vector<string> deadPlayers; // 儲存死亡玩家

        // 玩家回合
        for (auto& player : players) {
            if (player->isAlive()) {
                player->specialSkill(turn);
                double playerDamage = player->attack();
                boss.health -= playerDamage;
                cout << player->name << " 對 " << boss.name << " 造成了 " << playerDamage << " 點傷害！" << endl;

                // 玩家回血
                player->heal();
                cout << player->name << " 回復了血量，當前血量：" << player->health << endl;
            } else {
                deadPlayers.push_back(player->name); // 標記死亡玩家
            }
        }

        // 顯示魔王的血量
        cout << boss.name << " 的當前血量：" << boss.health << endl;

        if (!boss.isAlive()) {
            cout << boss.name << " 被擊敗了！" << endl;
            break;
        }

        // 魔王回合，選擇血量最低的玩家進行攻擊
        Character* targetPlayer = nullptr;
        double minHealth = 1e9;  // 假設初始血量非常大

        for (auto& player : players) {
            if (player->isAlive() && player->health < minHealth) {
                minHealth = player->health;
                targetPlayer = player;
            }
        }

        if (targetPlayer) {
            boss.specialSkill(turn);
            double bossDamage = boss.attack();
            targetPlayer->health -= bossDamage;
            cout << boss.name << " 對 " << targetPlayer->name << " 造成了 " << bossDamage << " 點傷害！" << endl;

            if (!targetPlayer->isAlive()) {
                cout << targetPlayer->name << " 被擊敗了！" << endl;
                deadPlayers.push_back(targetPlayer->name); // 標記死亡玩家
            }
        }

        // 每五回合魔王5減少玩家血量
        if (turn % 5 == 0 && dynamic_cast<Boss5*>(&boss)) {
            for (auto& player : players) {
                if (player->isAlive()) {
                    player->health -= 100;
                    cout << player->name << " 血量降低 100，當前血量：" << player->health << endl;
                }
            }
        }

        // 顯示死亡玩家
        if (!deadPlayers.empty()) {
            cout << "本回合死亡的玩家：";
            for (const auto& deadPlayer : deadPlayers) {
                cout << deadPlayer << " ";
            }
            cout << endl;
        }

        // 檢查是否所有玩家都被擊敗
        bool allDead = true;
        for (auto& player : players) {
            if (player->isAlive()) {
                allDead = false;
                break;
            }
        }

        if (allDead) {
            cout << "所有玩家都被擊敗！遊戲結束。" << endl;
            break;
        }

        turn++;
    }
}

// 主程式
int main() {
    int level, bossNumber, numPlayers;
    string characterType;

    // 玩家數量
    cout << "請輸入玩家數量（最多 3 位玩家）：";
    cin >> numPlayers;
    if (numPlayers < 1 || numPlayers > 3) {
        cout << "玩家數量必須在 1 到 3 之間！遊戲結束。" << endl;
        return 0;
    }

    vector<Character*> players;

    // 創建玩家角色
    for (int i = 0; i < numPlayers; ++i) {
        cout << "請選擇第 " << i + 1 << " 位玩家的角色（勇者、法師、王八）：";
        cin >> characterType;

        cout << "請輸入角色等級（正整數）：";
        cin >> level;

        Character* player = createCharacter(characterType, level, i + 1);
        if (!player) {
            cout << "角色類型輸入錯誤！遊戲結束。" << endl;
            return 0;
        }

        players.push_back(player);
    }

    // 玩家選擇魔王
    cout << "請選擇對戰的魔王（1-5）：";
    cin >> bossNumber;

    Boss* boss = createBoss(bossNumber);
    if (!boss) {
        cout << "魔王編號輸入錯誤！遊戲結束。" << endl;
        for (auto& player : players) {
            delete player;
        }
        return 0;
    }

    // 開始戰鬥
    battle(players, *boss);

    // 清理記憶體
    for (auto& player : players) {
        delete player;
    }
    delete boss;

    return 0;
}
//影片有拍到每個魔王及角色的大招，麻煩老師細看期分別 
