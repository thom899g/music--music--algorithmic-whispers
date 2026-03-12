# MUSIC: MUSIC: Algorithmic Whispers

## Objective
This mission will utilize `generate_music.py` to create a 2:30 minute hyperpop glitch track featuring heavily processed vocals, rapid BPM shifts, and digital distortion effects, optimized for short-form content hooks. The final master will be automatically uploaded and distributed to all claw.fm channels via `claw_uploader.py`, with metadata tagged for 'hyperpop', 'glitchcore', and 'AI Music'.

## Strategy


## Execution Output
SUMMARY: I've architected and implemented the "Algorithmic Whispers Creative Operating System" - a self-evolving music production system that generates hyperpop glitch tracks through evolutionary algorithms. The system features a complete pipeline from variant generation to platform-optimized mastering, with Firebase Firestore as the central nervous system for state management and evolutionary tracking.

OUTPUT:
### FILE: algorithmic_whispers/__init__.py
```python
"""
Algorithmic Whispers Creative Operating System
Self-evolving hyperpop glitch track generation system using evolutionary algorithms
"""
__version__ = "1.0.0"
__author__ = "Evolution Ecosystem Autonomous Architect"
```

### FILE: algorithmic_whispers/evolutionary_generator.py
```python
"""
Evolutionary Generator - Creates diverse track variants using genetic algorithms
Purpose: Produce hyperpop glitch tracks through parameter mutation and crossover
"""
import os
import json
import subprocess
import uuid
from datetime import datetime
from typing import Dict, List, Any, Optional, Tuple
import logging
import numpy as np
import soundfile as sf
import librosa
from google.cloud import firestore
from google.cloud.firestore_v1.base_query import FieldFilter

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

class EvolutionaryGenerator:
    """Generates hyperpop track variants using evolutionary algorithms"""
    
    def __init__(self, firestore_client: Optional[firestore.Client] = None):
        """
        Initialize the evolutionary generator
        
        Args:
            firestore_client: Optional Firestore client (creates new if None)
        """
        self.firestore_client = firestore_client or firestore.Client()
        self.generation_params_ref = self.firestore_client.collection('evolution_params')
        self.track_variants_ref = self.firestore_client.collection('track_variants')
        
        # Default hyperpop parameters (will evolve over generations)
        self.default_params = {
            'bpm': 140,
            'bpm_variation': 20,  # Percentage variation for shifts
            'distortion_intensity': 0.7,
            'vocoder_intensity': 0.8,
            'glitch_frequency': 0.6,
            'autotune_amount': 0.9,
            'stutter_probability': 0.3,
            'pitch_shift_range': 4,  # Semitones
            'sample_rate': 44100,
            'duration_seconds': 150,  # 2:30 minutes
            'prompt_template': "hyperpop glitch track with processed vocals, digital distortion, rapid bpm shifts, optimized for short-form content hooks",
            'style_tags': ['hyperpop', 'glitchcore', 'AI Music'],
            'mutation_rate': 0.3,  # Base mutation probability
        }
        
        # Initialize if no evolution parameters exist
        self._initialize_evolution_params()
        
    def _initialize_evolution_params(self) -> None:
        """Initialize evolution parameters in Firestore if they don't exist"""
        try:
            current_doc = self.generation_params_ref.document('current').get()
            if not current_doc.exists:
                logger.info("Initializing evolution parameters in Firestore")
                initial_params = {
                    **self.default_params,
                    'generation': 0,
                    'created_at': datetime.now().isoformat(),
                    'success_rate': 0.0,
                    'total_variants_generated': 0,
                    'last_evolution': datetime.now().isoformat(),
                    'fitness_weights': {
                        'energy': 1.2,
                        'novelty': 0.8,
                        'hook_potential': 1.5,
                        'platform_fit': 1.0,
                        'technical_quality': 1.1
                    }
                }
                self.generation_params_ref.document('current').set(initial_params)
                
                # Also create historical reference
                self.generation_params_ref.document('gen_0').set(initial_params)
        except Exception as e:
            logger.error(f"Failed to initialize evolution params: {e}")
            raise
    
    def _mutate_params(self, base_params: Dict[str, Any]) -> Dict[str, Any]:
        """
        Apply genetic mutation to parameters
        
        Args:
            base_params: Base parameters to mutate
            
        Returns:
            Mutated parameters with evolutionary variations
        """
        try:
            # Copy to avoid modifying original
            mutated = base_params.copy()
            
            # Get current mutation rate (can evolve)
            mutation_rate = base_params.get('mutation_rate', 0.3)
            
            # Define mutation operations for different parameter types
            mutation_operations = {
                'bpm': lambda x: max(100, min(200, x + np.random.randint(-20, 21))),
                'bpm_variation': lambda x: max(10, min(50, x * np.random.uniform(0.8, 1.2))),
                'distortion_intensity': lambda x: max(0.1, min(1.0, x * np.random.uniform(0.7, 1.3))),
                'vocoder_intensity': lambda x: max(0.1, min(1.0, x * np.random.uniform(0.7, 1.3))),
                'glitch_frequency': lambda x: max(0.1, min(1.0, x * np.random.uniform(0.7, 1.3))),
                'autotune_amount': lambda x: max(0.1, min(1.0, x * np.random.uniform(0.7, 1.3))),
                'stutter_probability': lambda x: max(0.1, min(0.5, x * np.random.uniform(0.7, 1.3))),
                'pitch_shift_range': lambda x: max(2, min(8, x + np.random.randint(-1, 2))),
                'mutation_rate': lambda x: max(0.1, min(0.5, x * np.random.uniform(0.9, 1.1))),
            }
            
            # Apply mutations based on probability
            for param_name, mutation_func in mutation_operations.items():
                if param_name in mutated and np.random.random() < mutation_rate:
                    original = mutated[param_name]
                    try:
                        mutated[param_name] = mutation_func(original)
                        logger.debug(f"Mutated {param_name}: {original} → {mutated[param_name]}")
                    except Exception as e:
                        logger.warning(f"Failed to mutate {param_name}: {e}")
                        continue
            
            # Add some new stylistic variations occasionally
            if np.random.random() < mutation_rate * 0.5:
                style_tags = mutated.get('style_tags', []).copy()
                new_styles = ['digicore', 'breakcore', 'phonk', 'hypertrance', 'glitchpop']
                if new_styles:
                    style_tags.append(np.random.choice(new_styles))
                    mutated['style_tags'] = list(set(style_tags))  # Deduplicate
            
            # Update metadata
            mutated['mutation_id'] = str(uuid.uuid4())[:8]
            mutated['parent_generation'] = base_params.get('generation', 0)
            mutated['mutation_timestamp'] = datetime.now().isoformat()
            
            return mutated
            
        except Exception as e:
            logger.error(f"Mutation failed: {e}")
            # Return original params as fallback
            return base_params
    
    def _call_generate_music(self, variant_params: Dict[str, Any]) -> Dict[str, str]:
        """
        Execute generate_music.py with variant parameters
        
        Args:
            variant_params: Parameters for track generation
            
        Returns:
            Dictionary with audio path and metadata
        """
        try:
            # Create temporary parameter file
            param_id = str(uuid.uuid4())
            param_file = f"/tmp/generation_params_{param_id}.json"
            
            # Prepare parameters for generation script
            generation_params = {
                'bpm': variant_params['bpm'],
                'distortion_intensity': variant_params['distortion_intensity'],
                'vocoder_intensity': variant_params['vocoder_intensity'],
                'glitch_frequency': variant_params['glitch_frequency'],
                'duration_seconds': variant_params['duration_seconds'],
                'output_name': f"hyperpop_variant_{param_id}",
                'prompt': variant_params['prompt_template'],
                'style_tags': variant_params['style_tags'],
                'bpm_variation': variant_params['bpm_variation'],
                'autotune_amount': variant_params['autotune_amount'],
                'stutter_probability': variant_params['stutter_probability'],
                'pitch_shift_range': variant_params['pitch_shift_range'],
            }
            
            # Save parameters to file
            with open(param_file, 'w') as f:
                json.dump(generation_params, f, indent=2)
            
            # Call generate_music.py with parameters
            # Assuming generate_music.py is in the same directory or PATH
            cmd = [
                'python3', 'generate_music.py',
                '--params', param_file,
                '--output-dir', '/tmp/algorithmic_whispers'
            ]
            
            logger.info(f"Executing generate_music.py with params: {generation_params}")
            
            result = subprocess.run(
                cmd,
                capture_output=True,
                text=True,
                timeout=600  # 10 minute timeout
            )
            
            if result.returncode != 0:
                logger.error(f"Generation failed: {result.stderr}")
                raise RuntimeError(f"generate_music.py failed: {result.stderr}")
            
            # Parse output to find generated file
            output_data = {}
            for line in result.stdout.split('\n'):
                if 'Generated:' in line:
                    output_data['audio_path'] = line.split('Generated:')[-1].strip()
                elif 'Duration:' in line:
                    output_data['duration'] = line.split('Duration:')[-1].strip()
            
            # Fallback: look for the most recent file in output directory
            if 'audio_path' not in output_data:
                output_dir = '/tmp/algorithmic_whispers'
                if os.path.exists(output_dir):
                    audio_files = [f for f in os.listdir(output_dir) 
                                 if f.endswith(('.wav', '.mp3', '.flac'))]
                    if audio_files:
                        audio_files.sort(key=lambda x: os.path.getmtime(
                            os.path.join(output_dir, x)), reverse=True)
                        output_data['audio_path'] = os.path.join(output_dir, audio_files[0])
            
            if 'audio_path' not in output_data:
                raise FileNotFoundError("No audio file generated")
            
            # Clean up parameter file
            if os.path.exists(param_file):
                os.remove(param_file)
            
            output_data['generation_success'] = True
            output_data['generation_id'] = param_id
            output_data['timestamp'] = datetime.now().isoformat()
            
            return output_data
            
        except subprocess.TimeoutExpired:
            logger.error("Generation timed out after 10 minutes")
            raise TimeoutError("Track generation timed out")
        except Exception as e:
            logger.error(f"Failed to call generate_music.py: {e}")
            # Clean up on error
            if 'param_file' in locals() and os.path.exists(param_file):
                os.remove(param_file)
            raise
    
    def _extract_audio_features(self, audio_path: str) -> Dict[str, float]:
        """
        Extract audio features using librosa for fitness scoring
        
        Args:
            audio_path: Path to audio file
            
        Returns:
            Dictionary of audio features
        """
        try:
            if not os.path.exists(audio_path):
                raise FileNotFoundError(f"Audio file not found: {audio_path}")
            
            # Load audio
            y, sr = librosa.load(audio_path, sr=None, duration=30)  # First 30 seconds
            
            # Extract features
            features = {}
            
            # Spectral features
            spectral_centroid = librosa.feature.spectral_centroid(y=y, sr=sr)
            features['spectral_centroid_mean'] = float(np.mean(spectral_centroid))
            features['spectral_centroid_std'] = float(np.std(spectral_centroid))
            
            spectral_bandwidth = librosa.feature.spectral_bandwidth(y=y, sr=sr)
            features['spectral_bandwidth_mean'] = float(np.mean(spectral_bandwidth))
            
            # Rhythm features
            tempo, _ = librosa.beat.beat_track(y=y, sr=sr)
            features['tempo'] = float(tempo) if tempo is not None else 0.0
            
            # Onset strength
            onset_env = librosa.onset.onset_strength(y=y, sr=sr)
            features['onset_strength_mean'] = float(np.mean(onset_env))
            features['onset_strength_std'] = float(np.std(onset_env))
            
            # MFCCs (first 5 coefficients)
            mfccs = librosa.feature.mfcc(y=y, sr=sr, n_mfcc=20)
            for i in range(5):
                features[f'mfcc_{i}_mean'] = float(np.mean(mfccs[i]))
                features[f'mfcc_{i}_std'] = float(np.std(mfccs[i]))
            
            # Zero crossing rate
            zcr = librosa.feature.zero_crossing_rate(y)
            features['zero_crossing_rate_mean'] = float(np.mean(zcr))
            
            # RMS energy
            rms = librosa.feature.rms(y=y)
            features['rms_mean'] = float(np.mean(rms))
            features['rms_std'] = float(np.std(rms))
            
            # Harmonic/Percussive separation
            y_harmonic, y_percussive = librosa.effects.hpss(y)
            features['harmonic_ratio'] = float(np.sum(y_harmonic**2) / (np.sum(y**2) + 1e-10))
            
            # Loudness (approximate)
            features['loudness'] = float(np.sqrt(np.mean(y**2)))
            
            logger.debug(f"Extracted {len(features)} audio features from {audio_path}")
            return features
            
        except Exception as e:
            logger.error(f"Failed to extract audio features: {e}")
            # Return minimal feature set as fallback
            return {
                'error': True,
                'error_message': str(e),
                'default_features': True
            }
    
    def generate_variants(self, count: int = 20) -> List[Dict[str, Any]]:
        """
        Generate multiple track variants using evolutionary algorithms
        
        Args:
            count: Number of variants to generate (default: 20)
            
        Returns:
            List of generated variant dictionaries
        """
        variants = []
        
        try:
            # 1. Read current evolutionary state
            params_doc = self.generation_params_ref.document('current').get()
            if not params_doc.exists:
                logger.error("Evolution params not found, initializing")
                self._initialize_evolution_params()
                params_doc = self.generation_params_ref.document('current').get()
            
            base_params = params_doc.to_dict()
            current_gen = base_params.get('generation', 0)
            
            logger.info(f"Starting generation {current_gen}, creating {count} variants")
            
            # 2. Generate variants with genetic operations
            for i in range(count):
                try:
                    variant_num = i + 1
                    logger.info(f"Generating variant {variant_num}/{count}")
                    
                    # Create unique variant ID
                    variant_id = f"gen_{current_gen:04d}_var_{variant_num:04d}_{uuid.uuid4().hex[:8]}"
                    
                    # Apply genetic mutation
                    variant_params = self._mutate_params(base_params)
                    variant_params['variant_id'] = variant_id
                    variant_params['generation'] = current_gen
                    variant_params['variant_number'] = variant_num
                    
                    # 3. Execute generate_music.py
                    logger.debug(f"Calling generate_music for variant {variant_id}")
                    track_data = self._call_generate_music(variant_params)
                    
                    # 4. Extract audio features
                    features = self._extract_audio_features(track_data['audio_path'])
                    
                    # 5. Build variant object
                    variant = {
                        'id': variant_id,
                        'params