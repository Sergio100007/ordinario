import time
import random
from typing import Dict, List, Any

class LeaderboardSystem:
    def __init__(self):
        # Simulamos una base de datos persistente con resultados de equipos
        self.db_submisions: List[Dict[str, Any]] = [
            {"equipo": f"Equipo_{i}", "puntuacion": random.randint(10, 100)} 
            for i in range(1, 51)
        ]
        # Variables para la caché en memoria del leaderboard
        self.cached_leaderboard: List[Dict[str, Any]] = []
        self.last_cache_update: float = 0.0
        self.cache_duration: float = 30.0  # Expira cada 30 segundos como pide el enunciado

    def _rebuild_leaderboard_heavy_calculation(self) -> List[Dict[str, Any]]:
        """
        Simula una operación pesada de CPU/IO que lee de la base de datos,
        ordena los datos y reconstruye el ranking de los participantes.
        """
        # Simulación de un retraso de procesamiento leve (ej. agregación o consulta DB)
        time.sleep(0.05) 
        sorted_teams = sorted(self.db_submisions, key=lambda x: x["puntuacion"], reverse=True)
        return [{"rank": idx + 1, **team} for idx, team in enumerate(sorted_teams)]

    def get_leaderboard_WITHOUT_cache(self) -> List[Dict[str, Any]]:
        """Enfoque ineficiente: Cada petición de un espectador recalcula todo."""
        return self._rebuild_leaderboard_heavy_calculation()

    def get_leaderboard_WITH_cache(self) -> List[Dict[str, Any]]:
        """Enfoque eficiente: Devuelve la caché o la reconstruye si pasaron 30s."""
        current_time = time.time()
        # Si la caché expiró o es la primera llamada, se reconstruye
        if current_time - self.last_cache_update > self.cache_duration or not self.cached_leaderboard:
            self.cached_leaderboard = self._rebuild_leaderboard_heavy_calculation()
            self.last_cache_update = current_time
        return self.cached_leaderboard

if __name__ == "__main__":
    print("=== PRUEBA DE RENDIMIENTO: SISTEMA DE LEADERBOARD ===")
    system = LeaderboardSystem()
    
    # Número de peticiones concurrentes simuladas de espectadores móviles
    NUM_PETICIONES = 100
    print(f"Simulando {NUM_PETICIONES} accesos de espectadores móviles concurrentes...\n")
    
    # -------------------------------------------------------------
    # MEDICIÓN 1: Sin Estrategia de Caché
    # -------------------------------------------------------------
    start_time = time.perf_counter()
    for _ in range(NUM_PETICIONES):
        _ = system.get_leaderboard_WITHOUT_cache()
    end_time = time.perf_counter()
    duration_no_cache = end_time - start_time
    print(f"[1] TIEMPO SIN CACHÉ (Recalculando siempre): {duration_no_cache:.4f} segundos")
    
    # -------------------------------------------------------------
    # MEDICIÓN 2: Con Estrategia de Caché (Snapshot de 30 segundos)
    # -------------------------------------------------------------
    start_time = time.perf_counter()
    for _ in range(NUM_PETICIONES):
        _ = system.get_leaderboard_WITH_cache()
    end_time = time.perf_counter()
    duration_with_cache = end_time - start_time
    print(f"[2] TIEMPO CON CACHÉ (Respetando ciclo de 30s): {duration_with_cache:.4f} segundos")
    
    # -------------------------------------------------------------
    # Conclusiones del Benchmark Técnico
    # -------------------------------------------------------------
    mejora = (duration_no_cache / duration_with_cache) if duration_with_cache > 0 else float('inf')
    print("\n=== CONCLUSIÓN TÉCNICA ===")
    print(f"La arquitectura con caché en memoria es aprox. {mejora:.1f} veces más rápida.")
    print("Esto evita que el endpoint colapse ante la avalancha de espectadores móviles en las últimas 12h.")
