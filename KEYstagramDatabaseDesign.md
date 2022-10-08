# KEYstagram 데이터베이스 설계 실습

## 절차

> 2022.10.5(수)

1. MySQL 설치
2. 사용자 생성 및 권한 부여

> 2022.10.6(목)

3. 종이/펜 + ERD cloud로 데이터베이스 밑그림 그림
![20221006 SEB_BE_41 강은영 인스타그램 스키마_erd cloud](https://user-images.githubusercontent.com/87472526/194678639-182d265e-023b-47a5-9b3f-5194af2500d8.png)

> 2022.10.7(금)

4. MySQL Workbench 설치 및 Connection

5. 설계 수정 및 기능 추가해서 ERD 그리며 데이터베이스 개념/논리적 모델링 수행

  - 추가 기능
    - 대댓글 달기
    - 댓글 및 대댓글 좋아요
    - 사용자, 포스트, 댓글, 대댓글 신고

![20221007 SEB_BE_41 강은영 인스타그램 스키마_workbench v2](https://user-images.githubusercontent.com/87472526/194678738-72ca0099-adf6-4fe5-ac04-4f1197a140a3.png)

### MySQL Workbench Forward Engineering 결과 SQL 스크립트
```sql
-- MySQL Workbench Forward Engineering

SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0;
SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0;
SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';

-- -----------------------------------------------------
-- Schema KEYstagram
-- -----------------------------------------------------

-- -----------------------------------------------------
-- Schema KEYstagram
-- -----------------------------------------------------
CREATE SCHEMA IF NOT EXISTS `KEYstagram` DEFAULT CHARACTER SET utf8 ;
USE `KEYstagram` ;

-- -----------------------------------------------------
-- Table `KEYstagram`.`user`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`user` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`user` (
  `user_no` INT NOT NULL AUTO_INCREMENT,
  `user_id` VARCHAR(45) NOT NULL,
  `pwd` VARCHAR(45) NOT NULL,
  `email` VARCHAR(100) NOT NULL,
  `inviting_user_no` INT NULL,
  `date` DATETIME NOT NULL,
  `status` CHAR(1) NOT NULL DEFAULT 'Y',
  PRIMARY KEY (`user_no`),
  INDEX `fk_user_user1_idx` (`inviting_user_no` ASC) VISIBLE,
  CONSTRAINT `fk_user_user1`
    FOREIGN KEY (`inviting_user_no`)
    REFERENCES `KEYstagram`.`user` (`user_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `KEYstagram`.`post`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`post` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`post` (
  `post_no` INT NOT NULL AUTO_INCREMENT,
  `user_no` INT NOT NULL,
  `description` VARCHAR(3000) NULL,
  `date` DATETIME NOT NULL,
  `link` VARCHAR(300) NOT NULL,
  PRIMARY KEY (`post_no`),
  INDEX `fk_posts_users_idx` (`user_no` ASC) VISIBLE,
  CONSTRAINT `fk_posts_users`
    FOREIGN KEY (`user_no`)
    REFERENCES `KEYstagram`.`user` (`user_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `KEYstagram`.`image`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`image` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`image` (
  `image_no` INT NOT NULL AUTO_INCREMENT,
  `post_no` INT NOT NULL,
  `origin_name` VARCHAR(300) NOT NULL,
  PRIMARY KEY (`image_no`),
  INDEX `fk_images_posts1_idx` (`post_no` ASC) VISIBLE,
  CONSTRAINT `fk_images_posts1`
    FOREIGN KEY (`post_no`)
    REFERENCES `KEYstagram`.`post` (`post_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `KEYstagram`.`hashtag`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`hashtag` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`hashtag` (
  `hashtag_no` INT NOT NULL AUTO_INCREMENT,
  `hashtag` VARCHAR(300) NOT NULL,
  PRIMARY KEY (`hashtag_no`))
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `KEYstagram`.`comment`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`comment` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`comment` (
  `comment_no` INT NOT NULL AUTO_INCREMENT,
  `post_no` INT NOT NULL,
  `user_no` INT NOT NULL,
  `comment` VARCHAR(1000) NOT NULL,
  `date` DATETIME NOT NULL,
  PRIMARY KEY (`comment_no`),
  INDEX `fk_comments_users1_idx` (`user_no` ASC) VISIBLE,
  INDEX `fk_comments_posts1_idx` (`post_no` ASC) VISIBLE,
  CONSTRAINT `fk_comments_users1`
    FOREIGN KEY (`user_no`)
    REFERENCES `KEYstagram`.`user` (`user_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_comments_posts1`
    FOREIGN KEY (`post_no`)
    REFERENCES `KEYstagram`.`post` (`post_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `KEYstagram`.`comment_on_comment`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`comment_on_comment` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`comment_on_comment` (
  `comment_on_comment_no` INT NOT NULL AUTO_INCREMENT,
  `comment_no` INT NOT NULL,
  `user_no` INT NOT NULL,
  `comment` VARCHAR(1000) NOT NULL,
  `date` DATETIME NOT NULL,
  PRIMARY KEY (`comment_on_comment_no`),
  INDEX `fk_comments_on_comment_comments1_idx` (`comment_no` ASC) VISIBLE,
  INDEX `fk_comments_on_comment_users1_idx` (`user_no` ASC) VISIBLE,
  CONSTRAINT `fk_comments_on_comment_comments1`
    FOREIGN KEY (`comment_no`)
    REFERENCES `KEYstagram`.`comment` (`comment_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_comments_on_comment_users1`
    FOREIGN KEY (`user_no`)
    REFERENCES `KEYstagram`.`user` (`user_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `KEYstagram`.`join_hashtag`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`join_hashtag` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`join_hashtag` (
  `hashtag_no` INT NOT NULL,
  `post_no` INT NOT NULL DEFAULT 0,
  `comment_no` INT NOT NULL DEFAULT 0,
  `comment_on_comment_no` INT NOT NULL DEFAULT 0,
  PRIMARY KEY (`hashtag_no`, `post_no`, `comment_no`, `comment_on_comment_no`),
  INDEX `fk_join_comment_hashtag_hashtag1_idx` (`hashtag_no` ASC) VISIBLE,
  INDEX `fk_join_comment_hashtag_comment_on_comment1_idx` (`comment_on_comment_no` ASC) VISIBLE,
  INDEX `fk_join_comment_hashtag_post1_idx` (`post_no` ASC) VISIBLE,
  CONSTRAINT `fk_join_comment_hashtag_comment1`
    FOREIGN KEY (`comment_no`)
    REFERENCES `KEYstagram`.`comment` (`comment_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_join_comment_hashtag_hashtag1`
    FOREIGN KEY (`hashtag_no`)
    REFERENCES `KEYstagram`.`hashtag` (`hashtag_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_join_comment_hashtag_comment_on_comment1`
    FOREIGN KEY (`comment_on_comment_no`)
    REFERENCES `KEYstagram`.`comment_on_comment` (`comment_on_comment_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_join_comment_hashtag_post1`
    FOREIGN KEY (`post_no`)
    REFERENCES `KEYstagram`.`post` (`post_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `KEYstagram`.`like`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`like` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`like` (
  `user_no` INT NOT NULL,
  `post_no` INT NOT NULL DEFAULT 0,
  `comment_no` INT NOT NULL DEFAULT 0,
  `comment_on_comment_no` INT NOT NULL DEFAULT 0,
  PRIMARY KEY (`user_no`, `post_no`, `comment_no`, `comment_on_comment_no`),
  INDEX `fk_like_user1_idx` (`user_no` ASC) VISIBLE,
  INDEX `fk_like_comment1_idx` (`comment_no` ASC) VISIBLE,
  INDEX `fk_like_comment_on_comment1_idx` (`comment_on_comment_no` ASC) VISIBLE,
  CONSTRAINT `fk_like_post1`
    FOREIGN KEY (`post_no`)
    REFERENCES `KEYstagram`.`post` (`post_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_like_user1`
    FOREIGN KEY (`user_no`)
    REFERENCES `KEYstagram`.`user` (`user_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_like_comment1`
    FOREIGN KEY (`comment_no`)
    REFERENCES `KEYstagram`.`comment` (`comment_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_like_comment_on_comment1`
    FOREIGN KEY (`comment_on_comment_no`)
    REFERENCES `KEYstagram`.`comment_on_comment` (`comment_on_comment_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `KEYstagram`.`follow`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`follow` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`follow` (
  `following_user_no` INT NOT NULL,
  `followed_user_no` INT NOT NULL,
  PRIMARY KEY (`following_user_no`, `followed_user_no`),
  INDEX `fk_follow_user2_idx` (`followed_user_no` ASC) VISIBLE,
  CONSTRAINT `fk_follow_user1`
    FOREIGN KEY (`following_user_no`)
    REFERENCES `KEYstagram`.`user` (`user_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_follow_user2`
    FOREIGN KEY (`followed_user_no`)
    REFERENCES `KEYstagram`.`user` (`user_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


-- -----------------------------------------------------
-- Table `KEYstagram`.`report`
-- -----------------------------------------------------
DROP TABLE IF EXISTS `KEYstagram`.`report` ;

CREATE TABLE IF NOT EXISTS `KEYstagram`.`report` (
  `reporting_user_no` INT NOT NULL,
  `reported_user_no` INT NOT NULL DEFAULT 0,
  `post_no` INT NOT NULL DEFAULT 0,
  `comment_no` INT NOT NULL DEFAULT 0,
  `comment_on_comment_no` INT NOT NULL DEFAULT 0,
  `reason` VARCHAR(1000) NOT NULL,
  `report_date` DATETIME NOT NULL,
  `status` VARCHAR(1) NOT NULL DEFAULT 'N',
  `resolution_date` DATETIME NULL,
  PRIMARY KEY (`reporting_user_no`, `reported_user_no`, `post_no`, `comment_no`, `comment_on_comment_no`),
  INDEX `fk_report_post1_idx` (`post_no` ASC) VISIBLE,
  INDEX `fk_report_user2_idx` (`reported_user_no` ASC) VISIBLE,
  INDEX `fk_report_comment1_idx` (`comment_no` ASC) VISIBLE,
  INDEX `fk_report_comment_on_comment1_idx` (`comment_on_comment_no` ASC) VISIBLE,
  CONSTRAINT `fk_report_user1`
    FOREIGN KEY (`reporting_user_no`)
    REFERENCES `KEYstagram`.`user` (`user_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_report_post1`
    FOREIGN KEY (`post_no`)
    REFERENCES `KEYstagram`.`post` (`post_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_report_user2`
    FOREIGN KEY (`reported_user_no`)
    REFERENCES `KEYstagram`.`user` (`user_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_report_comment1`
    FOREIGN KEY (`comment_no`)
    REFERENCES `KEYstagram`.`comment` (`comment_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION,
  CONSTRAINT `fk_report_comment_on_comment1`
    FOREIGN KEY (`comment_on_comment_no`)
    REFERENCES `KEYstagram`.`comment_on_comment` (`comment_on_comment_no`)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION)
ENGINE = InnoDB;


SET SQL_MODE=@OLD_SQL_MODE;
SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS;
SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS;
```
